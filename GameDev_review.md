# Bitewise operators

- **smallest** unit of data.
- bool take **8** bits.
- When treat each bit in a data type as it own value(bit value): **bit field**.
- Consider several bits within a bit field instead of just one: **bit-masking**.
- **& ^ ~ << >>**
- Flip a certain bit: **number ^= 1UL(*unsinged long*) << n**
- Checking a certain bit: **(number >> n) & 1UL**
- Checking a mask: wasMaskOn = **((number) & mask == mask)**
- Check polarity of two float is the same or not: **bitwise check first bit**

# Binary / Hexadecimal Numbers

- $2^6 = 64$, $2^8 = 256$

- max signed int32: $2^{31}-1$(1 bit will be used to represent the sign, *-1* for 0)
- Min signed int32: $-2^{31}$

- Represent a negative number in binary/hexadecimal:
  - Write down it's absoulute value in binary(int8)
  - takle 1's **complement**(revert 1/0)
  - +1 (convert into 2's complement)
  - (if hexadecimal needed) transfer into hexadecimal
- HEX_1 - HEX_2 == **HEX_1 + (2's complement of)HEX_2**
- **0b** means binary
- **0x** means hexadecimal
- ASCII table has *127* entries, A is *065*
- *Big-edian* means store start with the most significant bit.

# Floating Point Numbers

![image-20220711171549222](/Users/deneb/Library/Application Support/typora-user-images/image-20220711171549222.png)

- Exponent is **128 - value** because when value is large number, exponent will be negative and that allows **fractions**

- **significand / mantissa** [1, 2), which means these 8 bits only represents *decimals* after 1, to save one significand digit.

- decimals in binary:

  ```
  while(decimals != 0){
  		decimals *= 2;
  		decimals = decimals >= 1 ? decimals-1 : decimals;
  		res.push_back(decimals >= 1 ? 1 : 0);
  }
  ```

  Which will lead some decimals cannot be represented by binaries accurately (those decimal digits cannot be eliminated by $*2$).

- This will cause compare in float is not accurate sometimes.
- **FLI_EPSILON** used as a minimal gap of float numbers while comparing. (equals specifically)

# Data Type Sizes

| Data Type |   32-bit size    | 62-bit size (if different) |
| :-------: | :--------------: | :------------------------: |
|   bool    |  1 byte(8 bits)  |                            |
|   char    |  1 byte(8 bits)  |                            |
|   short   | 2 bytes(16 bits) |                            |
|    int    | 4 bytes(32 bits) |                            |
|   long    | 4 bytes(32 bits) |      8 bytes(64 bits)      |
|   float   | 4 bytes(32 bits) |                            |
|  double   | 8 bytes(64 bits) |                            |
|  Pointer  | 4 bytes(32 bits) |      8 bytes(64 bits)      |

# Alignment

For c++ compiler:

- Add padding before allocate next data if current offset is not a integer times of the next data size.

- Add padding after all data are allocated, if current data block size is not a integer times of the **biggest data size**.

  ```c++
  Struct MixedData{
    	char charData;		 //1 bytes
  		double doubleData; //8 bytes
  		int intData;			 //4 bytes
  };
  ```

  Size of above struct should be: 1 + (current offset is not integer times of next data size double *8 bytes*, so add a 7 bytes padding here)7 + 8 + 4 + 2(current struct size is 22 and is not integer times of biggest data size 8, so add 2 padding here to make it 24) = 24.

# Caches

## Memory layout

memory usage:

- **Code**: The compiled program.
- **Globals**: Store global variables.
- **Heap**: Store dynamically allocated memory.(pointer pointed memory)
- **Stack:** Store statically allocated memory. (parameters/ local variables)

## To avoid memory fragmentation

- Relocatable Heap

  ![image-20220716190324415](/Users/deneb/Library/Application Support/typora-user-images/image-20220716190324415.png)

  ![image-20220716190349561](/Users/deneb/Library/Application Support/typora-user-images/image-20220716190349561.png)

## PS4 CPU layout

![image-20220716193052264](/Users/deneb/Library/Application Support/typora-user-images/image-20220716193052264.png)

## Cache line

Least bytes cache will load from memory (**64** on PS4).

- ```U32 g_jobCount[6]``` works bad because size of a **U32** is less than a cache line, all data will be shared among cores and cause cores fighting.

- ```c++
  Struct JobCount{
  		U32 m_count;
    	U8 m_padding[60];
  };
  JobCount g_JobCount[6]
  ```

- Above works better because it makes sure a single core is calling all the jobcount data, keep the 26 cycles calling efficiency.

# Virtual methods

- **Static dispatch**: for normal in-calls methods, compiler will create a routine for each of them. (unique for each class, shared by all entities). When a function was called, compiler will find the generated routine of this function, and call it.
- **Dynamic dispatch**: call via a base class pointer, try to call a derived class function.This cannot be dispatched in compiling (since compiler don't know that kind of entity was pointed by the base class pointer). It have to find the right reference at runtime.
- **vtable**:(for every class contains virtual functions), contains an entry of each virtual function (including inherited and overrided functions), and points to its definition.
  - if overrided, subclass's entry will point to their own definition
  - if not, will point to base class's definition.
  - Shared by all instances
- **vpointer**: a pointer exist in each **instance** that points to the corresponding vtable. 
  - Stored in each instance.
  - When a virtual function of this instance is called:
    - call virtual pointer
    - found correct virtual table
    - call correct function

# Virtual classes

- Diamond inheritance
- ```Class B : virtual public A{}```
- Virtual inheritance manually specified that the derived class will **Share** the status of base class.
- Basically means for class B and classes derived from class B, they all share the same entity of class A.

# Virtual Destructor

- Virtual destructor is needed unless it's not designed to be inherited.

# Pure virtual

- ```virtual int foo(int x) = 0;```
- In the virtual table, the entry of a pure virtual function will be null.

# Static

- Inside functions

  - ```c++
    void counter(){
    	static int count = 0;
    }
    ```

  - **count** will hold its value through function calls

  - variable will be stored on global area, instead of stack.

  - If not initialized, will be automatically initialized by 0  (only appiled to **primitive** types, used defined types will apply their own constructor)

- Static data member in class

  - ```c++
    Class A{
    public:
    	static int count = 0;
    };
    ```

  - Shared by all objects.

  - Must be initialized manually.

  - Cannot be redefined.

- Static member functions

  - ```c++
    Class A{
    public:
    	static void foo();
    }
    ```

  - Cannot access to non-static data/functions.

  - Cannot be declared **const**, **virtual**, **volatile**.

  - Can be called through ```A::foo();```

# Friend

- Friend class

  - ```c++
    Class A{
    private:
    	int x;
    public:
    	friend class B; // now class B has access to A's private/protected variables/functions.
    }
    ```

- Friend functions

  - ```c++
    Class A{
    private:
    	int x;
    public:
    	friend void setVariable(A a){
        this->x = a.x;
      } // now function setVariable() can access to all A's private/protected variables/functions
    }
    ```

- *Friend* **cannot** be inherited, or passed. (friend's friend is not friend)

# Const

- When its decorating a member function, means it cannot change any member variables in the class.

- ```c++
  possiblepos1 int possiblepos2  * possiblepos3 myptr
  ```

  - If const in *possiblepos1* or *possiblepos2*, it's a const value which means **\*myptr** is const, and ```*myptr = val``` will cause an error.
  - If const in possiblepos3, it's a const pointer which means **myptr** itself is const, and ```myptr = &val``` will cause an error.

# Explicit

- Used before class constructors to prevent implicit conversions in classes.

- ```c++
  Class A{
  	int x;
  	(explicit)A(int in_x) {
  		 x = in_x;
  	}
  }
  
  A a = 1;
  //above codes will cause implicit conversion without explicit as:
  //A temp(1);
  //a = temp;
  //~temp;
  ```

# Extern

- used when call a function/variable havn't been defined yet.
- delay "looking for definition" to linking stage.
- ```Extern "C" {void foo();}```: compile *foo()* in C (instead of C++).

# Inline

- **suggest** a function should be inlined.
- Directly copy function body instead of function call.
- More quick, more space cost.
- **No recursive, no loop**
- All functions defined **in class** are inline functions.

# Union

```c++
union A{
	int a;
	struct {int x, int y, int z}b;
};
```

- All member variable share the same memory.
- When one of them is defined, others will turn *undefined*. Override begin with the first byte in the memory.

```C++
A a;
a.b.x = 3;
a.b.y = 4;
a.b.z = 5;
a.a = 6;
cout << a.b.x << a.b.y << a.b.z <<endl;
// here will get 6 4 5, since the first 4 bytes was overrided by new definition.
```

- Size is the same as the biggest data in the union.

# New operator

```c++
type *myPtr = new(address) (type)(init)
```

For example, ```int *ptr = new(&x) int(10);``` means define a **int** pointer that stored in the address of **x**, abd init it by **10**.

# std::move()

```c++
Swap(int &a, int &b){
	int temp = std::move(a);
	int a = std::move(b);
	int b = std::move(temp);
} // std::move() was used to swap resources(instead of copy them)
```

# Include guard

- Prevent a header file to be included and compiled multiple times in referecing.

- ```c++
  #pragma once
  ```

- ```c++
  #ifndef HEADER_H_
  #def HEADER_H_
  // codes here
  #endif
  ```

# include VS using

- include basically **copy** the value of the file into the include line.
- includes was used to aware part of your source code(usually .c/.cpp file) of the declearation fo other part of source code (usually .h/.hpp file)
- using basically tell the compiler you're using something, declear the scope so that you don't need to do it at every call.

# Allocation pairings

- C-style allocation
  - **malloc(size_t size)**
    - won't call constructor of target object.
    - will return *void
  - **free(void *ptr)**
- C++-style allocation
  - **new**, **new[]**
    - will call constructor of target object.
    - will return type of target.
  - **delete**, **delete[]**

# Time complexity

$n! > 2^n > n^2 >> n log n >> n > log n > 1$

# Smart pointers

- ```std::unique_ptr``` when only need one reference of the object
- ```std::shared_ptr``` when need multiple reference of a object, will be released automatically when reference count is 0
- ```std::weak_ptr``` hold a reference of a **shared_prt**, but won'n increase the reference count.
