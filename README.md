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

