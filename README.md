# embedded-software-advanced-coursera
## Pointers
`data_type * pointer_name;` 
data_type says how much data that we want to access. 
size of the pointers will be same because they are addresses but not the dereferencing. 

### Null pointers
used to check for valid pointer. `uint32_t *ptr;` <- this will have a garbage data
```
#define NULL ((void*)0)
uint32_t * ptr=NULL;
```
### Struct pointers
```
typedef struct foo{
  uint8_t A;
  uint8_t B;
  uint8_t C
} foo_t;
foo_t S;
foo_t * S_ptr=&S;

```
dereferencing these struct pointers as given below
`S_ptr->varB;`
### Pointers casting
```
uint32_t *ptr3=(uint32_t *)0x100;
uint8_t *ptr1 = (uint8_t *)ptr3;
```
### function pointers
```
void (*foo)();
int8_t void (*bar)(int8_t a, int8_t *b);

//example of calling function
(*foo)(); or foo();
```
## Compiler attributes
### alignmnet must be power of two.
`in8_t foo __attribute__((aligned(4)));`
now the foo can be stored in 0x100, 0x104, 0x108 ans son on. earlier 0x100, 0x101, 0x102, ...

### alignment on structure
```
typedef struct {
  int8_t a;
  int8_32_t b;
  int8_t c;
} str1;

```
now `sizeof(str1)=6 bytes`
```
typedef struct {
  int8_t a __attribute__((aligned(4)));
  int8_32_t b__attribute__((aligned(4)));
  int8_t c__attribute__((aligned(4)));
} str2;
```
now `sizeof(str2)=12 bytes`
```
typedef struct {
  int8_t a ;
  int8_32_t b;
  int8_t c;
} str3 __attribute__((aligned));
```
now `sizeof(str3)=16 bytes` because take bigger components type and take the value to find the closest value which is power of two.

### packed
```
typedef struct {
  int8_t a ;
  int8_32_t b;
  int8_t c;
} str4 __attribute__((packed));
```
here `sizeof(str4)=6 bytes`

### inline
this will copy the content to the line which caolls the function rather than open a new stack.
```
__attribute__((always_inline)) inline int32_t add(int32_t x, int32_t y){
  return (x+y);
}

```
without the always_inline compiler may ignore if needed. 

### function pragmas
add extra options for compilation on that particular area.

```
#pragma GCC push
#prgma GCC optimize("O0")
int32_t add(int32_t x, int32_t y)
{
  return(x+y);
}
#pragma GCC ppop
```
### note
always remeber these are for gcc. so use compile time switch for them. to avoid errors when you use other compilers.
```
#ifndef (__GNUC__)
#define __attribute__(x)
#endif
```
## Register definition files
directly dereference memory
![image](https://github.com/user-attachments/assets/90d533ed-abe7-43d8-948c-f3cbf7566465)
``(*((volatile uint16_t *)0x4000000)) = 0x0202;``
but not readle or maintable

so use preprocessor definitions
```
#define HWREG8(x) (*((volatile uint8_t *)(x)))

#define TA0CTL (HWREG16(0x4000000))

TA0CTL=0x202; //example usgae

```
use compile time switch to include correct register definition file. 
volatile -> not to optimize the code.

### structure overlay
```
typedef struct{
  __IO uint16_t CTL;
  __IO uint16_t CCTL[7];
  __IO uint16_t R;
  __IO uint16_t CCR[7];
  __IO uint16_t EX0;
  uint16_t RESERVED[6];
  __I uint16_t IV;
} Timer_A_Type;

#define __IO (volatile)
#define __I (volatile coinst)

/*define the base peripheral*/
#define PERIPH_BASE  ((uint32_t)0x4000000)
#define TIMER_A0_BASE (PERIPH_BASE + 0x0000000)
#define TIMER_A1_BASE (PERIPH_BASE + 0x0000400)

/*multiple timers different addresses*/
#define TIMER_A0 ((Timer_A_Type *) TIMER_A0_BASE)

//example usage
TIMER_A0->CTL = 0x0202;

```
## bit manipulation
```
set 4th bit: *ptr |= 0x10;
clear 4th bit: *ptr &= ~(0x10);
toggle 4th bit: *ptr ^= 0x10;
```
use these mask in preprocessor for more readable code.
`#define MASK1 (0x10)` or `*ptr &= ~(BIT6 | BIT7);`

## data structures
### structures
```
struct <struct name> {
  <type> <member name>;
};

```
### enumerations
```
enum <enum_type>{
  <enum_name>= <enum_value>, //assigned constatnt value
  <enum-name>, //auto assigned values
}


//example
enum month m = APRIL;
```
naming convention is to use _e like Error_e.
when compiling use `-fshort-enums` to shorten the enum type size

### unions
multiple members occupy same address.
```
typedef union {
  uint8_t x;
  uint32_t y;
} ex1;
```
it's equal to largest member. 
usecase when you use for a shared memory. 
## encapsulation
when using the struct then pass them in pointer to a function not individual paramerets to avaoid the overhead of copying those values when calling a function. 
to record data.
`void gather_sample(WeatherSample **w);`
to read data.
`float get temp(weatherSample *w);`

example for encapsulation:
```
typedef struct {
  uint32_t x;
  uint32_t y;
  uint32_t z;
  time_t t;
} Position_t;

typedef struct {
  Position_t pos[MAX];
  uint32_t count;
} Movement_t;

// usage
Movement_t ball;
Position_t b1;
b1.x=0x12;
b1.y=0x42;
b1.z=0x22;
b1.t=time(NULL);

ball.position[ball.count]=b1;

ball.count++;
```
## bit fields
allows to read&write within a union or struct that is smaller than a byte. 
```
struct <name>{
  uint32_t w:BITSIZE;
  uint32_t x:BITSIZE;
};
```
![image](https://github.com/user-attachments/assets/8446c1b8-2c02-4c45-bdae-d1bd1e17521b)
## LIFO buffer
```
//store the status of the buiffer
typedef enum {
	LB_NO_ERROR =0,
	LB_FULL,
	LB_NOT_FULL,
	LB_EMPTY,
	LB_NOT_EMPTY,
	LB_NULL,  
} LB_Status;

//two ways for structures
// 1.
typedef struct{
  <type> length;
  <type> *base;
  <type> count; 
} LIFO_Buf_t;

// 2.
typedef struct{
  uint32_t length;
  uint8_t *base;
  uint8_t *head;
} LIFO_Buf_t;

// instance buffer
LIFO_Buf_t lbuf;

//set length parameter
lbuf.base = (uint8_t *) malloc(length);
if(! lbuf.base ){ return LB_ERROR; }
lbuf.length = length;
//set head pointer
lbuf.head=lbuf.base;

/*need to check full before adding */
LB_Status LIFO_Is_Buf_Full (LIFO_Buf_t * lbuf){
	/*check if pointers are valid*/
	if(! lbuf || ! lbuf->head || ! lbuf->base ){
		return LB_NULL;
	}
	if((lbuf->head-lbuf->base)=>lbuf->length){
	  return LB_FULL;
	}
	else {
		return LB_NOT_FULL;
	}
}
/*add items*/
LB_Status LIFO_Add_Item(LIFO_Buf_t * lbuf, uint8_t item){
	/*check if pointers valid*/
	LB_Status status = LIFO_Is_Buf_Full(lbuf);
	if (status == LB_NOT_FULL){
		*lbuf->head = item;
		lbuf->head++;
		Status = LB_NO_ERROR;
	} 
	return Status;
}

```
## Circular buffer

```
typedef enum {
	CB_NO_ERROR =0;
	CB_FULL;
	CB_NOT_FULL;
	CB_EMPTY;
	CB_NOT_EMPTY;
	CB_NULL;
} CB_Status;

//implement struct
typedef struct {
	uint8_t *base;
	uint8_t *head;
	uint8_t *tail;
	uint32_t length;
	uint32_t count; //optional
} CB_t;
// using count it is easy to check the full
if (buf.length == buf.count) {
	return CB_FULL;
}

CB_Status CB_is_buffer_full(CB_t * cbuf){
	/*check the pointers are valid*/
	if (! cbuf || ! cbuf->head || ! cbuf->tail || !cbuf->base){
		return CB_NULL;
	}
	if ( (cbuf->tail == cbuf->head+1) || (cbuf->head == cbuf->tail + (cbuf->length -1))){
		return CB_FULL;
	}
	else {
		return CB_NOT_FULL;
	}
}

CB_Status CB_is_add_item(CB_t * cbuf, uint8_t item){
	/*check the pointers are valid*/
	CB_Status status = CB_is_buffer_full(cbuf);

	if (status == CB_NOT_FULL){
		*cbuf->head = item;
		// increment head
		if (cbuf->head == cbuf->head + (cbuf->length -1)){
			cbuf->head= cbuf->base;
		}
		else{
			cbuf->head++;
		}
		status = CB_NO_ERROR;
	}
	return status;
}

```
## linked list
```
typedef struct {
	Node_t *next;
	uint32_t data;
} Node_t;
//add items
LL_Append(Node_t * node, uint32_t data){
	Node_t * nn =node;
	if (node ==NULL){
		return;
	}
	if (node->next ==NULL){
		node->next = (Node_t *)malloc(sizeof(Node_t));
		nn = node->next;
	}
	else {
		while( nn->next != NULL){
			nn = nn->next;
		}
		nn->next = (Node_t *)malloc(sizeof(Node_t));
		nn=nn->next;
	}
	nn->data = data;
	nn->next = NULL;
}

```






























