# CSA02-1925260337-Assignment5
This repository contains C Programming Assignment 5 (CO5), covering operators, decision-making statements, looping constructs and algorithm optimization. It includes program solutions, pseudocode, execution outputs, test cases and complexity analysis for Questions 1–7.
CO5 ASSIGNMENT 5 – ALL PSEUDOCODES AND C CODES
B. Hasini | Register No. 1925260337

==============================================================================
TASK A1 – DECOMPOSE THE PROGRAM INTO FUNCTIONS
==============================================================================
A1.1 FUNCTION BREAKDOWN
1. register_donor: validates and appends a donor record.
2. check_eligibility: checks whether the donor meets the donation interval and other rules.
3. find_donors: searches donors of a requested blood group.
4. update_stock: adds or subtracts units for a blood group.
5. save_donors / load_donors: writes and reads donor records in a text file.
6. save_stock / load_stock: writes and reads fixed-size stock records in a binary file.
7. issue_units: verifies stock, updates the stock record and reports success/failure.
8. print_report: displays eligible donors and current stock.

STRUCTURE CHART
main
├── register_donor
│   ├── validate_donor
│   └── append_donor
├── find_donors
│   └── check_eligibility
├── issue_units
│   └── update_stock
├── save_donors / load_donors
├── save_stock / load_stock
└── print_report

A1.2 FUNCTION PROTOTYPES AND PARAMETER DECISIONS
typedef struct { int day, month, year; } Date;
typedef struct {
    int id; char name[60]; char group[4]; int age;
    char phone[16]; Date last_donation;
} Donor;
typedef struct { char group[4]; int units; } Stock;

int register_donor(DonorList *list, const Donor *candidate);
int check_eligibility(const Donor *donor, Date today);
int find_donors(const Donor *donors, size_t count, const char *group,
                Date today, int *youngest_id, size_t *eligible_count);
int update_stock(Stock *stock, size_t count, const char *group, int delta);
int save_donors(const char *filename, const Donor *donors, size_t count);
int load_donors(const char *filename, DonorList *list);
int save_stock(const char *filename, const Stock *stock, size_t count);
int load_stock(const char *filename, Stock *stock, size_t capacity, size_t *count);
int issue_units(Stock *stock, size_t count, const char *group, int units);
void print_report(const Donor *donors, size_t count, Date today);

Pass small scalar values such as count, date fields and delta by value. Pass arrays/lists by pointer because C passes array addresses and copying a large collection is wasteful. Use const Donor * and const char * for input-only data so the function cannot modify it. Use DonorList * or Stock * when the function must update the caller's collection. Output parameters such as youngest_id and eligible_count are non-const pointers because the function writes results through them. File names are const char * because they are read-only strings.

A1.3 RETURNING TWO RESULTS
Alternative 1 – output pointers:
int find_eligible(const Donor *d, size_t n, const char *group,
                  Date today, size_t *count, int *youngest_id);
The function writes the eligible count and youngest donor ID through the last two parameters. Return a status code to indicate success/failure.

Alternative 2 – return a structure:
typedef struct { size_t count; int youngest_id; int found; } EligibilityResult;
EligibilityResult find_eligible_result(const Donor *d, size_t n,
                                       const char *group, Date today);
A structure groups related results and is convenient when more values are added. Pointer outputs avoid returning a larger object and are useful with status codes. Both scan n donors: O(n) time and O(1) extra space.

GLOBAL-VARIABLE ALTERNATIVE
A global DonorList and global result variables could be read or modified by functions without parameters. The asymptotic time and memory are unchanged: the same records are scanned and stored. However, functions become tightly coupled, tests depend on hidden state, reuse becomes harder, and simultaneous independent blood-bank instances become difficult. Parameter passing is preferred.

==============================================================================
TASK A2 – DESIGN WITH RECURSION
==============================================================================
A2.1 RECURSIVE PSEUDOCODE
DIGITAL_ROOT(n)
    IF n < 0 THEN n ← ABS(n)
    IF n < 10 THEN RETURN n                 // base case
    sum ← 0
    WHILE n > 0
        sum ← sum + (n MOD 10)
        n ← n DIV 10
    RETURN DIGITAL_ROOT(sum)                // recursive case

POWER_LINEAR(base, exp)
    IF exp = 0 THEN RETURN 1                // base case
    RETURN base × POWER_LINEAR(base, exp - 1)

POWER_FAST(base, exp)
    IF exp = 0 THEN RETURN 1                // base case
    half ← POWER_FAST(base, exp DIV 2)      // exactly one recursive call
    result ← half × half
    IF exp MOD 2 = 1 THEN result ← result × base
    RETURN result

A2.2 CALL TRACES
Digital root example: 9875 → 9+8+7+5=29 → 2+9=11 → 1+1=2.
Call chain: digital_root(9875) → digital_root(29) → digital_root(11) → digital_root(2) → return 2.
Deepest stack has four active calls (9875, 29, 11, 2).

POWER_LINEAR(2,4): P(2,4) → 2×P(2,3) → 2×P(2,2) → 2×P(2,1) → 2×P(2,0) → 16.
Calls recurrence: L(0)=1; L(n)=L(n−1)+1, so L(n)=n+1. Time O(n), stack O(n).

POWER_FAST(2,10): P(2,10) → P(2,5) → P(2,2) → P(2,1) → P(2,0), then returns 1024.
Calls recurrence: F(0)=1; F(n)=F(floor(n/2))+1, so F(n)=O(log n). Time O(log n), stack O(log n).

A2.3 RECURSION RISKS
Without a reachable base case, calls continue until stack resources are exhausted, commonly causing stack overflow. A loop is preferable when recursion depth may be large, when the recursive form gives no clarity benefit, or when constant stack use is important.

ALTERNATIVE DIGITAL ROOT
A loop repeatedly sums digits until one digit remains. For positive n, the direct digital-root formula is 0 when n=0, otherwise 1+(n−1) MOD 9. The loop is O(d) time for d digits and O(1) space; the formula is O(1) time and O(1) space. Recursive digit summing uses O(d) total digit work and O(log d) stack levels because each digit-sum reduces the number substantially.

==============================================================================
TASK A3 – DESIGN WITH POINTERS
==============================================================================
A3.1 POINTER EXPRESSIONS
Given int stock[8] = {40,25,12,8,30,18,7,14}; int *p = stock;
p + 3     : address of stock[3], not the stored value.
*(p + 3)  : value at stock[3], which is 8.
p[3]      : exactly equivalent to *(p + 3), value 8.
*p++      : evaluates the current pointed value (40), then advances p to stock[1]. Post-increment applies to p, not to the value.

MEMORY DIAGRAM (illustrative addresses)
Index:       0       1       2       3       4       5       6       7
Value:      40      25      12       8      30      18       7      14
Address:   1000    1004    1008    1012    1016    1020    1024    1028
p initially points to address 1000. p+3 points to 1012.

A3.2 RETURN POINTER TO LOWEST STOCK
int *lowest_stock(int stock[], size_t n) {
    if (stock == NULL || n == 0) return NULL;
    size_t min_index = 0;
    for (size_t i = 1; i < n; ++i)
        if (stock[i] < stock[min_index]) min_index = i;
    return &stock[min_index];
}
The returned address points into the caller-owned array, so the caller may update *result. Returning the address of a local variable is invalid because the local object's lifetime ends when the function returns; the pointer would dangle.

A3.3 COMMAND-LINE INTERFACE
bloodbank find O+
bloodbank issue A- 2
Validate argc before reading argv. Accept only known blood-group strings. For find, require argc == 3; for issue, require argc == 4 and a positive integer quantity. On invalid input, print usage to stderr and return a non-zero exit status. Check strtol's end pointer and range rather than using atoi.

ALTERNATIVE – RETURN INDEX
size_t lowest_stock_index(const int stock[], size_t n); return SIZE_MAX if n==0.
Caller: i=lowest_stock_index(stock,n); if(i!=SIZE_MAX) stock[i]=new_units.
Both approaches use O(n) time and O(1) extra space. A pointer is direct and convenient while the array remains at the same allocation. An index survives reallocating/moving the array, provided the logical ordering is unchanged; a pointer becomes invalid if realloc moves storage.

==============================================================================
TASK A4 – DESIGN DYNAMIC MEMORY USE
==============================================================================
A4.1 GROWABLE DONOR LIST PSEUDOCODE
ADD_DONOR(list, donor)
    IF list.size = list.capacity THEN
        IF list.capacity = 0 THEN new_capacity ← 2
        ELSE new_capacity ← 2 × list.capacity
        temp ← REALLOC(list.data, new_capacity × SIZEOF(Donor))
        IF temp = NULL THEN
            RETURN FAILURE                 // old list.data remains valid
        END IF
        list.data ← temp
        list.capacity ← new_capacity
    END IF
    list.data[list.size] ← donor
    list.size ← list.size + 1
    RETURN SUCCESS

Growth trace for 5 additions: capacity 2, 2, 4, 4, 8.

A4.2 MEMORY ERRORS
Memory leak: allocated memory is no longer reachable and is never freed. Example: overwriting list.data with a failed realloc result loses the original block. Prevention: assign realloc to a temporary pointer; free list.data once when finished.
Dangling pointer: pointer refers to an object whose lifetime has ended. Example: returning &local_donor from a function. Prevention: return a pointer to caller-owned/dynamic storage, or return a structure by value.
Double free: freeing the same allocation twice. Example: free(list.data) in both an error branch and common cleanup without setting it to NULL. Prevention: centralize cleanup and set freed pointers to NULL.

ALTERNATIVE – FIXED INCREMENT OF 10
Capacity becomes 10,20,30,... . With repeated growth by 10, the amount copied across n insertions is approximately 10+20+...+n = O(n²) in the worst case. Doubling copies a geometric series and gives O(n) total copying across n appends (amortized O(1) append). Fixed increments waste at most about 9 slots; doubling may leave nearly half the allocated capacity unused immediately after growth (less than the current size). Doubling is generally faster for large unpredictable data.

==============================================================================
TASK A5 – DESIGN THE FILE STORAGE
==============================================================================
A5.1 TEXT DONOR FILE AND BINARY STOCK FILE
DONOR TEXT FORMAT – one donor per line:
fprintf(fp, "%d|%s|%s|%d|%s|%02d-%02d-%04d\n", id, name, group,
        age, phone, day, month, year);
Text is human-readable, easy to inspect and more portable across machines. Validate and escape delimiter characters in names, and use fgets plus parsing rather than unsafe unbounded input.

STOCK BINARY FORMAT:
typedef struct { int group_code; int units; } Stock;
fwrite(stock, sizeof *stock, count, fp);
Binary fixed-size records are compact and support direct record positioning with fseek. Binary files may be machine/compiler dependent because of integer size, byte order and structure padding; use a defined serialization format if portability is required.

A5.2 UPDATE ONE BINARY RECORD PSEUDOCODE
UPDATE_STOCK_FILE(fp, index, new_units)
    IF fp is NULL OR index < 0 THEN RETURN FAILURE
    IF FSEEK(fp, index × SIZEOF(Stock), SEEK_SET) fails THEN RETURN FAILURE
    IF FREAD(record, SIZEOF(Stock), 1, fp) != 1 THEN RETURN FAILURE
    record.units ← new_units
    IF FSEEK(fp, -SIZEOF(Stock), SEEK_CUR) fails THEN RETURN FAILURE
    IF FWRITE(record, SIZEOF(Stock), 1, fp) != 1 THEN RETURN FAILURE
    IF FFLUSH(fp) fails THEN RETURN FAILURE
    RETURN SUCCESS

Text files have variable-length records. Changing one number may change line length and shift every following byte, so a simple fixed offset is not reliable. A text update normally reads records, changes the target and rewrites the file (often through a temporary file).

ALTERNATIVE – REWRITE TEXT FILE
Read all n records, update the matching group, write all n records to a replacement file, then replace the old file after successful completion. This is O(n) record processing per update and uses O(n) memory if all records are held at once (or O(1) record memory with a streaming temporary-file rewrite). Binary in-place update is O(1) record access when the index is known. Text files are usually larger due to digit separators and formatting; binary records are compact but less portable.

==============================================================================
TASK B1 – Instrumented Recursive Functions
==============================================================================

MAIN C CODE
#include <stdio.h>
long long calls;
long long power_linear(long long base, int exp) {
    calls++;
    if (exp == 0) return 1;
    return base * power_linear(base, exp - 1);
}
long long power_fast(long long base, int exp) {
    calls++;
    if (exp == 0) return 1;
    long long half = power_fast(base, exp / 2);
    long long result = half * half;
    if (exp % 2 != 0) result *= base;
    return result;
}
int digital_root(int n) {
    if (n < 0) n = -n;
    if (n < 10) return n;
    int sum = 0;
    while (n > 0) { sum += n % 10; n /= 10; }
    return digital_root(sum);
}
long long power_iterative(long long base, unsigned int exp) {
    long long result = 1;
    while (exp > 0) {
        if (exp & 1U) result *= base;
        base *= base;
        exp >>= 1U;
    }
    return result;
}
int main(void) {
    printf("digital_root(9875) = %d\n", digital_root(9875));
    calls = 0; printf("linear 2^30 = %lld, calls = %lld\n", power_linear(2,30), calls);
    calls = 0; long long fast = power_fast(2,30);
    printf("fast 2^30 = %lld, calls = %lld\n", fast, calls);
    int exponents[] = {10,100,1000,10000};
    for (int i=0;i<4;i++) {
        calls=0; (void)power_linear(1,exponents[i]);
        printf("linear exponent %d: calls = %lld\n",exponents[i],calls);
        calls=0; (void)power_fast(1,exponents[i]);
        printf("fast exponent %d: calls = %lld\n",exponents[i],calls);
    }
    printf("iterative 2^30 = %lld\n",power_iterative(2,30));
    return 0;
}

PSEUDOCODE
B1 PSEUDOCODE
DIGITAL_ROOT(n): if n<0 make positive; if n<10 return n; sum digits; recursively return DIGITAL_ROOT(sum).
POWER_LINEAR(base, exp): increment call counter; if exp=0 return 1; otherwise return base × POWER_LINEAR(base, exp−1).
POWER_FAST(base, exp): increment counter; if exp=0 return 1; half ← POWER_FAST(base, exp DIV 2) once; result ← half×half; if exp odd multiply by base; return result.
POWER_ITERATIVE(base, exp): result←1; while exp>0, multiply result by base when low bit is 1, square base, shift exp right; return result.
Test digital root 9875; test powers for exponent 30; count recursive calls for exponents 10,100,1000,10000 using base 1 to avoid integer overflow. The linear call count is exp+1; halving calls grow logarithmically.

ACTUAL OUTPUT
digital_root(9875) = 2
linear 2^30 = 1073741824, calls = 0
fast 2^30 = 1073741824, calls = 6
linear exponent 10: calls = 11
fast exponent 10: calls = 5
linear exponent 100: calls = 101
fast exponent 100: calls = 8
linear exponent 1000: calls = 1001
fast exponent 1000: calls = 11
linear exponent 10000: calls = 10001
fast exponent 10000: calls = 15
iterative 2^30 = 1073741824


==============================================================================
TASK B2 – Adversarial Inputs and Safe Input Handling
==============================================================================

MAIN C CODE
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdlib.h>
int valid_group(const char *g) {
    const char *groups[]={"A+","A-","B+","B-","AB+","AB-","O+","O-"};
    for (size_t i=0;i<8;i++) if (strcmp(g,groups[i])==0) return 1;
    return 0;
}
int main(int argc,char *argv[]) {
    char name[16];
    printf("Enter donor name (max 15 characters): ");
    if (!fgets(name,sizeof name,stdin)) { puts("No input received"); return 1; }
    name[strcspn(name,"\n")]='\0';
    if (strchr(name,'\0') && strlen(name)==sizeof name-1) {
        int ch; while ((ch=getchar())!='\n' && ch!=EOF) {}
        puts("Name was longer than the buffer; safely truncated/rejected.");
    }
    printf("Accepted input buffer: %s\n",name);
    if (argc==1) {
        fprintf(stderr,"Usage: %s find GROUP | issue GROUP UNITS\n",argv[0]);
    } else if (strcmp(argv[1],"find")==0 && argc==3 && valid_group(argv[2])) {
        printf("Valid find request for %s\n",argv[2]);
    } else if (strcmp(argv[1],"issue")==0 && argc==4 && valid_group(argv[2])) {
        char *end=NULL; errno=0; long units=strtol(argv[3],&end,10);
        if (errno || end==argv[3] || *end!='\0' || units<=0 || units>100000)
            puts("Invalid unit count");
        else printf("Valid issue request: %s %ld\n",argv[2],units);
    } else {
        puts("Invalid command or blood group");
    }
    FILE *fp=fopen("missing_bloodbank_demo.txt","r");
    if (!fp) perror("fopen missing_bloodbank_demo.txt");
    else fclose(fp);
    return 0;
}

PSEUDOCODE
B2 PSEUDOCODE
READ_NAME: use fgets into a fixed-size character array; if no input, report failure; remove newline; if buffer filled, drain remaining characters and report truncation/rejection.
VALID_GROUP(g): compare g against the eight supported blood groups; return true on match.
VALIDATE_COMMAND(argc,argv): if no arguments, print usage; for find require exactly GROUP; for issue require GROUP and a positive integer parsed by strtol; reject invalid groups, extra arguments and out-of-range quantities.
CHECK_FILE: attempt fopen; if it fails, call perror and continue safely without dereferencing a null FILE pointer.
Security note: an unbounded scanf into a small array can overflow it (undefined behaviour). Do not deliberately execute the unsafe version; demonstrate the contrast with bounded input and fgets.

ACTUAL OUTPUT
Enter donor name (max 15 characters): Name was longer than the buffer; safely truncated/rejected.
Accepted input buffer: VeryLongDonorNa
Invalid command or blood group
fopen missing_bloodbank_demo.txt: No such file or directory


==============================================================================
TASK B3 – Text File vs Binary File Experiment
==============================================================================

MAIN C CODE
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
typedef struct { int group_code; int units; } Stock;
static double elapsed(clock_t a,clock_t b){return (double)(b-a)/CLOCKS_PER_SEC;}
int main(void) {
    const int sizes[]={1000,10000,100000};
    for (int k=0;k<3;k++) {
        int n=sizes[k];
        FILE *ft=fopen("donors_demo.txt","w");
        FILE *fb=fopen("stock_demo.bin","wb");
        if(!ft||!fb){perror("fopen");return 1;}
        clock_t a=clock();
        for(int i=0;i<n;i++) fprintf(ft,"%d|%d\n",i+1,(i%50)+1);
        clock_t b=clock();
        for(int i=0;i<n;i++){Stock s={i+1,(i%50)+1};fwrite(&s,sizeof s,1,fb);}
        clock_t c=clock();
        fclose(ft);fclose(fb);
        long text_size,binary_size;
        ft=fopen("donors_demo.txt","r");fb=fopen("stock_demo.bin","rb+");
        fseek(ft,0,SEEK_END);text_size=ftell(ft);
        fseek(fb,0,SEEK_END);binary_size=ftell(fb);
        printf("n=%d text_bytes=%ld binary_bytes=%ld\n",n,text_size,binary_size);
        printf("write_text=%.6fs write_binary=%.6fs\n",elapsed(a,b),elapsed(b,c));
        rewind(ft);int id,units,read_count=0;char line[64];
        clock_t d=clock();
        while(fgets(line,sizeof line,ft)) if(sscanf(line,"%d|%d",&id,&units)==2)read_count++;
        clock_t e=clock();
        rewind(fb);Stock s;int binary_count=0;
        while(fread(&s,sizeof s,1,fb)==1)binary_count++;
        clock_t f=clock();
        printf("read_text=%.6fs read_binary=%.6fs records=%d/%d\n",elapsed(d,e),elapsed(e,f),read_count,binary_count);
        if(n==100000){
            /* Update record 50,000 (zero-based index 49,999) in place. */
            clock_t u=clock();
            if(fseek(fb,49999L*(long)sizeof(Stock),SEEK_SET)!=0 || fread(&s,sizeof s,1,fb)!=1)return 2;
            s.units=99;
            if(fseek(fb,-(long)sizeof s,SEEK_CUR)!=0 || fwrite(&s,sizeof s,1,fb)!=1)return 3;
            clock_t v=clock();
            printf("binary_update_record_50000=%.6fs\n",elapsed(u,v));
            /* Text equivalent: rewrite through a temporary file. */
            FILE *tmp=fopen("donors_temp.txt","w");
            if(!tmp)return 4;
            rewind(ft);int row=0;
            while(fgets(line,sizeof line,ft)){
                row++;
                if(row==50000)fprintf(tmp,"%d|99\n",row);
                else fputs(line,tmp);
            }
            fclose(tmp);
            clock_t w=clock();
            printf("text_rewrite_update_record_50000=%.6fs\n",elapsed(v,w));
        }
        fclose(ft);fclose(fb);
    }
    remove("donors_demo.txt");remove("stock_demo.bin");remove("donors_temp.txt");
    return 0;
}

PSEUDOCODE
B3 PSEUDOCODE
FOR n IN {1000,10000,100000}:
    OPEN text file and binary file; if either open fails, report error and stop.
    WRITE n records to text using fprintf; measure elapsed CPU time.
    WRITE n fixed-size Stock records to binary using fwrite; measure time.
    CLOSE and reopen files; obtain byte sizes using fseek/ftell.
    READ every text line using fgets and parse fields; count records and measure time.
    READ every binary record using fread; count records and measure time.
    IF n=100000: use fseek to record index 49999, fread one Stock, modify units, seek back and fwrite; measure.
    For text update, copy all lines to a temporary text file while replacing line 50000; measure rewrite.
    Print sizes, timings and record counts; close and remove temporary files.
Compare O(n) full read/write with O(1) indexed binary update when record index is known. Timings are machine-dependent.

ACTUAL OUTPUT
n=1000 text_bytes=6713 binary_bytes=8000
write_text=0.000092s write_binary=0.000028s
read_text=0.000261s read_binary=0.000022s records=1000/1000
n=10000 text_bytes=77094 binary_bytes=80000
write_text=0.000859s write_binary=0.000310s
read_text=0.000957s read_binary=0.000204s records=10000/10000
n=100000 text_bytes=870895 binary_bytes=800000
write_text=0.009771s write_binary=0.002797s
read_text=0.011366s read_binary=0.002311s records=100000/100000
binary_update_record_50000=0.000003s
text_rewrite_update_record_50000=0.005461s


==============================================================================
TASK B4 – Growable Donor List and Engineering Decision
==============================================================================

MAIN C CODE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
typedef struct { int id; char group[4]; } Donor;
typedef struct { Donor *data; int size, capacity; } DonorList;
int add_donor(DonorList *list,int id,const char *group) {
    if (!list || !group) return 0;
    if (list->size==list->capacity) {
        int new_capacity=(list->capacity==0)?2:list->capacity*2;
        Donor *p=realloc(list->data,(size_t)new_capacity*sizeof *list->data);
        if (p==NULL) return 0;
        list->data=p;
        list->capacity=new_capacity;
    }
    list->data[list->size].id=id;
    strncpy(list->data[list->size].group,group,3);
    list->data[list->size].group[3]='\0';
    list->size++;
    return 1;
}
int main(void) {
    DonorList list={NULL,0,0};
    const char *groups[]={"O+","A+","B-","AB+","O-"};
    for(int i=0;i<5;i++){
        if(!add_donor(&list,7001+i,groups[i])){
            fprintf(stderr,"Memory allocation failed\n");free(list.data);return 1;
        }
        printf("added %d (%s): size = %d, capacity = %d\n",
               list.data[i].id,list.data[i].group,list.size,list.capacity);
    }
    printf("allocated_bytes = %zu\n",(size_t)list.capacity*sizeof *list.data);
    free(list.data);
    list.data=NULL;list.size=0;list.capacity=0;
    puts("memory released");
    return 0;
}

PSEUDOCODE
B4 PSEUDOCODE
INITIALIZE list.data=NULL, list.size=0, list.capacity=0.
ADD_DONOR(list,id,group):
    IF size=capacity:
        new_capacity ← 2 if capacity=0 else capacity×2
        temp ← realloc(data,new_capacity×SIZEOF(Donor))
        IF temp=NULL: return failure without changing old data pointer
        data←temp; capacity←new_capacity
    Store id and a bounded, null-terminated blood group at data[size].
    size←size+1; return success.
MAIN: add five sample donors; after each addition print size and capacity; report allocated bytes; free data exactly once and clear fields.
Production decision: the supplied demo struct contains only id and group. A real eligibility report must load complete donor fields (including last-donation date and age) and apply eligibility rules. Keep all donors in memory only if measured memory fits the operating budget; otherwise process batches or maintain a sorted ID-to-file-offset index.

ACTUAL OUTPUT
added 7001 (O+): size = 1, capacity = 2
added 7002 (A+): size = 2, capacity = 2
added 7003 (B-): size = 3, capacity = 4
added 7004 (AB+): size = 4, capacity = 4
added 7005 (O-): size = 5, capacity = 8
allocated_bytes = 64
memory released


B1 REPEATED CALL EXPERIMENT CODE
#include <stdio.h>
long long calls;
long long power_bad(long long base,int exp){
    calls++;
    if(exp==0)return 1;
    return power_bad(base,exp/2)*power_bad(base,exp/2);
}
int main(void){
    calls=0;
    long long value=power_bad(1,10);
    printf("value=%lld calls=%lld\n",value,calls);
    return 0;
}
value=1 calls=31

Note: output screenshots are separate PNG files linked in the task list.
