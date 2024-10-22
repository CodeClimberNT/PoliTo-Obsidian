---
tags:
  - pds
  - Rust
  - s2
  - 2023-2024
---
[Rust tutorial exercise](https://github.com/rust-lang/rustlings/)
# Variabili
#2024-03-11
## Dinamiche

hanno un indirizzo assoluto determinato in fase di esecuzione. Tali variabili vengono allocate nella zona di memoria chiamata 'heap' e la sua allocazione/liberazione è a carico del programmatore.


## Allocazione di memoria
La libreria standard in C vi sono diverse funzioni per interagire con la memoria:
- `void *malloc(size_t s)`
	- memory allocation: permette di allocare uno spazio di memoria di dimensione s
	- in caso di fallimento ritorna NULL
- `void *calloc(int n, size_t s)`
	- clear allocation
	-  in caso di fallimento ritorna NULL
- `void *realloc(void *p, size_t s)`
	- reallocation, puntatore originale al primo blocco in memoria p, e nuova dimensione s
	- In caso di fallimento il puntatore p non viene toccato
- `free(...)`

In C++ esistono gli oggetti e nuovi metodi di allocazione
- `{}`
- `[]`

e di rilascio (dipende dal metodo di allocazione)
- `malloc(...) -> free(...)`
- `new -> delete`
- `new[num_elements] -> delete[]`

```cpp
#include <iostream>

class Test {
 public:
  Test() {
    std::cout << "Costruito Test @" << this << std::endl;
  }
  ~Test() {
    std::cout << "Distrutto Test @" << this << std::endl << std::endl;
  }
};

int main(){
	std::cout << "Main" << std::endl;
	Test t; // variabile locale
	std::cout << "Fine Main" << std::endl;
} // qui la variabile locale viene distrutta

/*
Output: 
Main
Costruito Test @<object_address>
Fine Main
Distrutto Test @<object_address>
*/
```

Un altro esempio
```cpp
#include <iostream>

class Test {
 public:
  Test() {
    std::cout << "Costruito Test @" << this << std::endl;
  }
  ~Test() {
    std::cout << "Distrutto Test @" << this << std::endl << std::endl;
  }
};

int main(){
	for(int i=0; i<3; i++){
		Test t; // variabile locale inizializzata
	} // qui la variabile locale viene distrutta
	//l'indirizzo in memoria dell'oggetto rimarrà sempre lo stesso
	//in quanto ad ogni iterazione avverrà una push e poi una pop
} 

```

esempio allocazione dinamica
```cpp
#include <iostream>

class Test {
 public:
  Test() {
    std::cout << "Costruito Test @" << this << std::endl;
  }
  ~Test() {
    std::cout << "Distrutto Test @" << this << std::endl << std::endl;
  }
};

int main(){
	Test *ptr = new Test(); //allocazione dinamica con un puntatore
	
	for(int i=0; i<3; i++){
		Test t; // variabile locale inizializzata
		//if(i==1) delete ptr; come liberare l'heap dal puntatore
	} // qui la variabile locale viene distrutta
	//l'indirizzo in memoria rimarrà sempre lo stesso
	//in quanto ad ogni iterazione avverrà una push e poi una pop
} //anche arrivando qui il puntatore non libera la memoria da Test,
// MEMORY LEAK!

```

## Puntatori
- permette l'accesso diretto ad un blocco di memoria
	- tale blocco può appartenere ad altri oggetti
	- `int A=10;`
	  `int* pA = &A;`
- Possono essere esplicitamente invalidi:
	- per convenzione con il valore `0`
- Quando si usano puntatori bisogna stabilire quali permessi sono associati al loro uso

### Ambiguità
- contiene indirizzo valido?
	- Un puntatore punta ad un byte, dal quale inizia un blocco di memoria, ma non definisce la sua validità. 
- Dimensione del blocco?
	- È impossibile avendo un puntatore al primo posto di un array, sapere quanto esso sia lungo e, quindi, evitare di andare out of bounds.
- Fino a quando è garantito l'accesso?
	- La liberazione di un blocco in memoria non rende in automatico tutti i puntatori disabilitati
- Si può modificare il contenuto?
	- Ovviamente si, ma bisogna fare attenzione al tipo di dato, dimensione del blocco e molto altro
- Occorre rilasciarlo?
	- Si, però bisogna determinare ***chi*** deve rilasciarlo.
		- Esiste il concetto di owner delle variabili
			- Se un puntatore viene associato ad una malloc si ha la responsabilità del rilascio
			- Se un puntatore punta solo ad una variabile non si ha la stessa responsabilità.
				- Quando la variabile esce out of scope il blocco di memoria verrà rilasciato
- Si può rilasciare o l'indirizzo è noto anche ad altri?

### Puntatori come metodo per accedere a blocchi di dati
Usato per accedere ad array monodimensionali
- Si perde di vista la dimensione dell'array
Importante non spostare il puntatore all'infuori dell'effettiva zona di appartenenza.

```c++
{ 
const char* ptr = "Quel ramo del lago di Como..."; 
 //conta gli spazi int n=0; 
 //usa l'aritmetica dei puntatori 
 for (int i=0; *(ptr+i)!=0; i++) { 
	 //usa il puntatore come fosse un array 
	 if ( isSpace(ptr[i]) ) n++; 
 } 
	 //altro... 
 }
```
### Puntatori per accedere a dati dinamici
Come metodo di accesso a dati memorizzati sullo heap però ci sono problemi:
- Quando viene rilasciato il puntatore?
- Quando viene rilasciato lo heap per permettere di riusare la memoria da parte del resto del sistema?

Esempio di implementazione
```c++
int* read_data() { 
	unsigned int size = count_available_data(); 
	if (size > 0 ) { 
	int* ptr= new int[size]; 
	consume_data(ptr, size); 
	//indica operazione eseguita 
	//correttamente return ptr; 
	} else //nessun dato disponibile 
	return nullptr; 
} 
int* result = read_data(); 
... 
if (result) delete[] result;
```

Altra implementazione tramite linked list. Come riconosco l'ultimo elemento?
> Il puntatore `*next` sarà uguale a `NULL` 

Nel caso di rilascio di una linked list bisogna rilasciare tutte le zone di memoria allocate. Se la linked list ha 10 elementi, bisogna liberare 10 blocchi di memoria, se 100 elementi liberare 100 blocchi etc.

In C++ sono stati introdotti gli smar pointer che permettono di sollevare il programmatore da alcune responsabilità:
- quando il ciclo di vita di un puntatore finisce anche il blocco di memoria da esso puntata sarà liberata
Esistono due tipi di smart pointer:
- unique: non può essere copiato
- shared: può essere copiato
	- tiene conto di quante copie esistono e solo quando l'ultimo puntatore finisce il suo ciclo di vita (non esistono più puntatori che puntano al blocco in memoria), il blocco di memoria verrà liberato

### Responsabilità del programmatore
- Limite di accesso
	- Spaziale
		- Non accedere a locazioni di memoria adiacenti
	- Temporale
		- non accedere ad un blocco di memoria al di fuori del suo ciclo di vita
- Assegnazione solo di indirizzi mappati 
- Rilasciare tutta la memoria dinamica allocata

### Rischi
- Accedere ad un indirizzo quando il corrispettivo ciclo di vita è terminato ha effetti non prevedibili
	- [[#Dangling Pointer]]
- Non rilasciare memoria inusata risulta in uno spreco di risorse
	- [[#Memory Leakage]]
	- saturazione spazio di indirizzamento
- Troppi release corrompe la memoria
	- [[#Double Free]]
- Assegnando un puntatore ad una locazione di memoria non mappato si genera una interruzione del processo
	- segmentation fault (S.O. termina il processo)
- Utilizzando un puntatore non inizializzato potrebbe puntare ovunque
	- Wild pointer
	- Violazoine di accesso
	- Corruzione area di memoria in uso da altri programmi

##### Dangling Pointer
```c++
{ 
	char* ptr = NULL; 
	{ 
		char ch='!'; 
		ptr = &ch; 
	} 
	printf("%c", *ptr); //leggi sotto
}
```
uscendo dalla parentesi graffa la variabile viene rimossa ma il suo valore rimane nello stack, quindi in teoria il puntatore in realtà punta ancora al valore di ch, il problema è che chiamando la funzione `printf` tutto lo stack pointer viene mosso sovrascrivendo lo spazio di memoria dove prima era scritto ch ed ora il ptr che punta ancora a quell'indirizzo darà un risultato inaspettatto (forse il valore di un registro o forse altro ancora)

##### Memory Leakage
```c++
{ 
	char* ptr = NULL; 
	ptr = (char*) malloc(10); 
	strncpy(ptr,10,"Leakage!"); 
	printf("%s\n", ptr); 
}
```

##### Double Free
```c++
{ 
	char* ptr1 = malloc(10); 
	char* ptr2 = ptr1; 
	// use ptr1 and/or ptr2 
	free(ptr1); 
	free(ptr2); 
}
```

### Pointer Management
#### Possession
#2024-03-12
Possession of a value: the one who allocate a block of memory is responsible to create a mechanism to later free that same space.
What happens if a pointer is copied? The possession is transferred? Who is then responsible to free that block of memory?
In #Rust it is possible to make a copy of a pointer, in that case it will happen a "movement": the address memory is changed. This is because if more pointer points to the same memory address, and one of them free up the heap, then the others pointers could cause unpredictable behavior.
To avoid this behavior the rust programming language take count of how many pointers point to an address and **only** when an address have no pointer associated with it the memory is free up by the garbage collector.
- this means that we could have many hole in the memory:
	- Pros: pointers will not cause unpredictable behavior
	- Cons: memory inefficiency

#### Dependency
When complex data structures are implemented (e.g. linked list), they use auxiliar memory space to function properly and store their data
In C++ the object `std::vector<T>` incapsulate a dynamic array of type generic T, here you can modify dynamically the data inside (also add new elements!). This object uses auxiliar memory block that need to be free up when the object is destroyed. 
Collectively this data type take the name of **dependency**.

##### How about write a good program?
It could be the easier approach, but how someone know if their code is 100% perfect? One of the many property human possess is the ability to make errors. We could almost never be sure that our program will not have a bug inside of it under some specific condition.

- Microsoft suggests to customers that used Windows XP to restart their machine at least once every day to reduce memory leak.
#### Survival Technics
Using the right tools to debug the program, will greatly reduce the memory bug.
Another options is to use a specific programming language that have this capabilities already built in
- Java, python, c# and more have garbage collector inside, this will avoid memory bug
- #Rust will make program more complex but with the ability to catch memory bugs at compile time

# Rust
## History
The creator of Rust (Graydon Hoare) was a #Mozilla employee and he hated C++ for all his problems with the memory and allegedly when he was stuck inside the elevator of his building and found out that the problem was a bug from C++ he, firstly as a personal project (2006), then sponsored by #Mozilla (2009), started developing the rust programming language with the specific goal to create a memory proof programming language. 
The language was so complex that he burn-out.
## Introduction
#Rust is a modern programming language aiming in correctness, speed and support on the current programming paradigm.
- it is achieved by not using garbage collector and without using any assumption in the data structure.

Rust has this name because is the programming language closest to the metal.
It is a static and heavily typed programming language (unlike #Python or #js, every variable has a type and is static), with different place where it can shine:
- Operating System
- Embedded
- Server application
- Browsers modules
- etc...

Different mechanism make the language without ambiguity:
- Type are know at compile time.
- Sophisticated inference mechanism to validate properties and context type to significantly reduce risk of error.  
- Total control of the memory usage with maximum optimization of the generated code to make the fastest and efficient code from the source one.


## Goal of the language
The main goal is to create a programming language without unpredictable behavior (very rare with the modern programming languages).
Offers abstraction with 0 cost with most of programming idiom.
- Abstraction like polymorphism in java need two pointers to keep tracks of what to do and is always used although sometimes the polymorphism is not needed
- Rust instead, keeps the cost of a program as low as possible, use just what is needed
Cargo is an environment specific for developing Rust.
The compiler is our friends: it's very explicit about the problem that let the code not to compile.

### Base assumption
#Java have a paradigm of compile once, run everywhere, but this create a lot of overhead as the bytecode need to come interpreted with the instruction set of the machine where it is ran, with the jvm
Rust as it's target is to be as close to the metal as possible cannot have this features.

> Rust is a very good language when considered with others languages with it's security benefits (main competitor is #Haskell) without sacrificing speed (as fast as #C) 
![[Pasted image 20240312122715.png]]

## Rust vs Other Languages
### Rust vs #C and C++


## Security

## Performance
The compiler optimize aggressively dimension and speed targeting a lot of cache uses.
 - If a small procedure is used, the compiler will encapsulate it inside the code to reduce space
## Modern Language
Generics, enum (often used)

## Security by Definition
==Possession==
Life span of a value, and transfer of possession from one function to another. Pointers checked at compilation time. Thread safety implemented in system type. **No hidden state**:
- Usual programming language: try and catch
	- The exception interrupt the execution and return to the caller, if the caller cannot handle the exception it will return to the caller of the caller and so on. This is an hidden state, as the program will be blocked at an unknow state during the run time
- Rust require to handle exception every time
	- The procedure will always return a result: `okay || where` 
		- okay: no exception raised
		- where: problem that must be handled

## Low level control
Rust will release variable when they end the their life:
- no garbage collector, so no downtime during the instruction execution
it is possible to run system call and much more

## Install Rust

Follow the instruction in [Rustup](ttps://rustup.rs)
There are different toolchain that can be used using windows gcc or others.

## Cargo
To create a package:
`cargo init <package_name>`
with this command in the current folder will be generate:
- `src` folder structure
	- default file: `main.rs`
- `.toml` file
To create a library
`cargo init --lib <library_name>`

To compile
`cargo build`

inside debug there will be the executable (Windows) / binary (Unix)

To clean the folders
`cargo clean`

To check
`cargo check`

Can add library with
`cargo add <library_name>`
## Syntax
`fn` to define a function
`println!` macro:
- if it ends with `!` it is called a macro
- In this case, from the family of `println` functions which is the more optimized! It depends on the arguments
	- e.g.: `println!("Hello World!");` optimized with this type passed and number of characters etc.

`usize` data type that change based on the machine it is:
- e.g. 64-bit cpu: data size 64 bit

## Language

### Variable and Types
#2024-03-15

Variables links a value to a name: introduced the keyword `let`. By default a variable is linked to a ***single value*** for the entire duration of its life. To indicate that a variable may be linked to a new value (of the same type) must be used also the new keyword `mut`
```rust
let v: i32 = 123; // v è immutabile e ha tipo i32 (intero a 32 bit con segno) 
// v = -5; // ERRORE: Non è possibile riassegnare il valore 

let mut w = v; // w può essere riassegnata, ha lo stesso tipo di v (i32) 
w = -5; // OK. Ora w vale -5 

let x = 1.3278; // x è immutabile di tipo f64 (floating point a 64 bit) 

let y = 1.3278f32; // y è immutabile di tipo f32 (floating point a 32 bit) 

let one_million = 1_000_000 // si possono usare ‘_’ per separare le cifre
```

In this case the variable is reused, unlike other programming languages (like python), rust is very conservative about memory usage!

### Block as expression
variable may be defined with block expression
```rust
fn main(){
	let y = {
		let x = 3;
		x + 1    //expression
		//ATTENTION NO ';'!
	};

	println!("The value of y is: {y}");
	//The value of y is: 4
}
```

The expression is the result of the block, and thus do not have the `;`, otherwise it would have been an instruction. 
It is possible to use `if` as expression
```rust
fn main(){
	let condition = true;
	let number = if condition { 5 } else { 6 };
	println!("The value of number is: {number}")
}
```

### Assignation

> C/C++ assignation return the value, so we can concatenate them to assign multiple variable
```C++
int a = b = c = 12;
```

> Rust does not permit this behavior as the principle of transitivity between type that it is not always true

### Elementary type
- Integer 
	- Signed
		- `i8, i16, i32, i64, i128, isize`
	- Unsigned
		- `u8, u16, u32, u64, u128, usize`
- Floating Point (IEEE 754)
	- `f32, f64`
- Boolean
	- `bool`
- Character (32-bit Unicode)
	- `char`
	- ==ATTENTION== String: array of char but with UTF-8 encoding. Dimension of string != num of character
- Unit
	- `()` - represent a tuple of 0 elements
	- Can be used as return expression as void from C/C++

### Char type

```rust
fn main() { 
	let c = 'z'; 
	let z = 'ℤ'; 
	let heart_eyed_cat = '😻'; 
	println!("The value of this char is: {heart_eyed_cat}"); 
}
```

## Traits
A trait describe a set of behavior (methods) that a specific type implements
Every type can implements from 0 to many traits (some may be predefined).

As an example, Int have some predefined like:
 - Copy
 - Eq
 - Ord

We already implemented the copy traits before:
```rust
let v: i32 = 123; 

let mut w = v;
```

Copying the value of v into the variable w is a trait!

### Dependence between traits

There are some trait without methods that are called marker

### Compilator Work
Lets see a code example and how it's checked
```rust
[...]

var1 < var2 
//the compiler will check that both var1 and var2 implements the Ord(PartialOrd) trait so that this operation can be performed!

[...]
```

## Type and traits
Sine difference type can implement common traits, there is a relation between all the type:
![[Pasted image 20240315110248.png]]

If a trait have `copy` then it will also implement `Clone` but the opposite is not always true. 
`Drop` is the trait that free up the memory. Deallocate from heap
==ATTENTION==
`Copy` exclude `Drop`!

There are around 20 predefined traits in rust.

## Tuple
Represent ordered collection of heterogeneous values.
Tuple has type (T1, T2, ..., Tn) where different Tx are different types.

```rust
let t: (i32, bool) = (123, false);   //t is made out of an integer 
									//and a boolean

let mut u = (3.14, 2.71);

u.0 = 0.0; //u = (0.0, 2.71)
```

## Pointers and Memory
- Reference
- Box
- Native Pointers

Unlike C/C++ the use of pointers is greatly simplified thanks to rust compiler that verify from a variable: 
- `ownership`
- `time of life`
	- Although this seems trivial as when the variable get out of scope, this is not always true, and the compiler check until the very last instruction from which that specific instruction is actually used
This verification make access very safe and let them happens only if it is possible to demonstrate the legitimacy of the action.
It is also possible to get things out of hands from the compiler and make compiler don't check for safety: `unsafe` block
But ***WHY***:
- using external library not written in rust.
- Knowing that some constraint about life of a variable or ownership are not needed in those block.

## Reference
Reference uses the keyword `&` and can be every value or expression
```rust
let r1 = &v;
let r2 = &123;
// although human sees it as a constant it is not! It's an immediate!
// the compiler knows the memory address in the stack and make a reference to it for r2
```
The code above define and initialize both the reference of `r1` and `r2`.
The variable `r1` **borrows** the value v and may access it (**read-only**) with the `*` operator.
The reference are **read-only** and can be assigned to others variable or passed as parameters to a function.
Until there is at ***least one*** ref the original value cannot be **modifiable** 

Expression `let r3 = &mut v;` (read as **RefMut**) define and initialize the mutable reference `r3`. This kind of reference let them change the value **but** this will block **any** other reference to be made! 
Read-Only during a write operation = **DATA HAZARD**

Although similar to pointers of C/C++, in rust they cannot be null, neither have an address of a released value or not yet inizialized.

Reference implement a logic of **single writer** or **multiple readers**
The compiler using a specific module called **borrow checker** will guarantee those constraints.

```rust
fn main() { 
	let mut i = 32; 
	let r = &i; 
	println!("{}", *r); 
	i = i+1; // Problem! r is still alive, thus cannot modify i
	println!("{}", *r); //if this println were omitted, the r pointer would have ended its life and the operation above could have happen!
} //deallocation is still done here! The compiler only knows when some operation may or may not happen based on the usage of variable

fn main() { 
	let mut i = 32; 
	let r = &mut i; 
	println!("{}", i);// Problem! cannot read!
	*r = *r+1; 
	println!("{}", *r); 
}
```

### Ref and RefMut
They are pointers without possession and can be derived from an already existing variable. Those who receive them have no responsibility around the release as it remains to the original owner

```rust
fn main() { 
	let v = 32; 
	let p = &v; 
	let pp = &p; 
	let ppp = &pp; 
	let str = ppp.to_string(); 
	println!("{str}"); 
}
```

### `Box<T>`
#2024-03-18
A local variable is ***always*** allocated into the stack
- it will expand of the dimension of the variable, no more, no less

to create a variable of type `Box<T>` it is used the construct: 
`let b = Box::new(v);` where `v` it's a value.
This instruction define a variable `b` that contains a pointer to a block allocated in the heap that will contains the value `v`
To access the value it is used the expression `*b`
Usage example:
```rust
fn useBox(){
	let i = 4;
	let mut b = Box::new((5,2));

	(*b).1 = 7; // access the second element of the tuple inside the box

	println!("{:?}", *b); //(5,7)
	println!("{:?}", b);  //(5,7)
} // when exiting the function the stack will contract and both i and b are deallocated
// i is an int and then no more operation are required
// b has the trait drop, then when b is deallocated, the heap also is freed up
```


Another example with a transfer of ownership:
```rust
fn makeBox(a: i32) -> Box<(i32, i32)>{
	let r = Box::new( (a, 1) );
	return r;
}

fn main(){
	// when r is returned, although the stack inside makeBox contracted and the local variable were deallocated, the reference to the Box<(i32, i32)> is transferred to b, and thus that block from the heap is not dropped 
	let /*mut*/ b = makeBox(5); //now b have ownership of the value Box<(i32, i32)>
	// the new owner can change the mutability if he wanted
	// b.0 = b.0 * 2;
	let c = b.0 + b.1;
}
```
### Print traits
- Display print a variable referenced with `{}` and is aimed to print to the final user
- Debug referenced with `{:?}` aimed for the programmer, the option `{:#?}` make a *pretty printing*
- Pointer `{:p}` is used to display the address of a pointer variable

### Copy vs Clone
The traits copy let duplication of values using `memcpy`. If the type implement only clone the operator `=` determines a movements of the value. To explicit copy a value without movement (*deep copy*) the method `.clone()` can be used
```rust
fn main() { 
	let x: u8 = 123; // u8 implements Copy 
	let y = x; // x can still be used 
	println!("x={}, y={}", x, y); 
	
	let v: Vec<u8> = vec![1, 2, 3]; // Vec<u8> implements Clone, but not Copy 
	let w = v; // This would move the value, rendering v unusable 
	println!("w={:?}", w); 

	let z = v.clone(); // This will make a copy making z and v indipendent

	v[0] = 10;
	z[1] = 50;
	
	println!("v={:?} z={:?}", v, z);
}
```
### Drop

### Movement

## Array
```rust
let a: [i32; 5] = [1, 2, 3, 4, 5]; //a array of 5 integers

let b = [0; 5]; // 5 integers of 0s

let l = b.len();

let v = [1, 2, 3, 4];
println!("{}", v[10]); // index out of bound - COMPILE TIME!

fn f (i: usize)->usize{
	return (i+10)
}

println!("{}", v[f(0)]); // index out of bound - run time PANIC!

```

## Slice
Rust make possible referencing a sequence of consecutive values from which the length is unknown at compile time,
- A slice of elements of type T (`&[T]`) it's a data type of two consecutive values: pointer at the start of the sequence and the numbers of elements in the sequence
- A slice it's create like a reference from a portion of an array or vec:
```rust
let a = [1,2,3,4];
let s1: &[i32] = &a; // s1 contiene i valori 1, 2, 3, 4
let s2 = &a[0..2]; // s2 contiene i valori 1, 2 
let s3 = &a[2..]; // s3 contiene i valori 3, 4

println!("{:?}",s1[1]); // 2 

let s4 = &mut a[...]; // this is the only way to change value of a slice
println!("{:?}",s4[0]); // 1
s4[0] = 10;
println!("{:?}",s4[0]); // 10
```


## `Vec<T>`
The type `Vec<T>` represent a resizable sequence of elements of type `T` allocated on the heap.
- There a different methods to access, insert or remove values from/into him
A `Vec<T>` variable is a tuple of three private values:
1. Pointer to a buffer allocated to the heap where the elements are stored
2. Unsigned integer with the overall dimension of the buffer
3. Unsigned integer with the total numbers of elements in the buffer

If a request to insert a new element in the `Vec<T>` object this element will be inserted in the first free position of the buffer 

```rust
fn useVec(){
	// although for the Box<T> there is no need to specify the type, here it is a must as it has to know the amount of memory needed for each allocation
	let mut v: Vec<i32> = Vec::new(); // allocate in the stack three elements: ptr, size, capacity [x,0,0]

	v.push(2); // [x,1,4]
	
	v.push(4); // [x,2,8]

	let mut s = &mut v; // <-slice, it will have a [ptr, size] = [x, 2]
	//there is a transfer of ownership and v is no longer accessible
	
	s[1] = 8; // [2, 8]
	
}
```

The capacity size every time the max size is reached double in size.
es. if start with 4 -> 8, when 8 byte are filled -> 16 etc.

The `Vec<T>` does not change it's size by itself if enough items are removed, this is the programmer job to do, and is done using the method `.shrink_to_fit();` 

==Attention==
If a `.pop();` is done (remove last item), the value are temporarily allocated into the stack before being destroyed.

`let a = v.remove(1);` using this method you can assign the removed item into the variable a. This will not invoke the trait `drop` from the element inside the `Vec<T>`!

## String
There are two keywords to define a string:
- `str`: immutable
- `String`: mutable

Those methods will define a string using the UTF-8 coding (**not a single byte**)
So a string is not an array of byte (\*char from C/C++) but an array of UTF-8!

### `str`
Since str are immutable they can be accessed via slice and ***cannot*** be manipulated
```rust
let s = "Ciao Mamma" == let s: &str = "Ciao Mamma"; 
```


> This type of data remain alive also after the end of the scope!
### `String`
Like a `Vec<T>` the keyword `String` define a data structure that will have:
- pointer
- size
- capacity

All the method available for the object `&str` are equally usable with the `&String` object

```rust
fn main(){
let hello = "hello,";

let mut s = String::new();

s.push_str(hello); // allocate in the heap enough space to allocate the hello string (copy the str array, not move!)

s.push_str(" world!"); // allocate more space in the heap to add this immediate string at the end of the current string
} // s deallocated with all the mutable value in the heap, "hello, " will still remain in memory
```

## Ref of byte `&[u8; N]`
`N` is optional and is the length of the array
```rust
fn main(){
let name: &[u8; 6] = b"Torino";
//the 'b' will ask to convert the string in byte 
println!("{:?}", name)
}
```

### Conversion `&[u8] -> str`
The method `String::from_utf8(my_vec)` will return two possible value:
- `Ok('my_str')`
- `Error...`

If the conversion return ok, using the method `.unwrap()` can be used to read the value inside 
```rust
String::from_utf8(my_vec).unwrap();
```

### `OsStr` and `OsString`
Operating system convention to string
- Linux uses UTF-8
- Windows uses 16bit char: Unicode Basic Multilingual Plane 0

To pass a string to the os the variable must be of type `osString/OsStr`

### `Path/PathBuf`
Linux and Windows different char for path
- Windows: `/` 
- Linux: `\`
The object `Path` will convert a string that contains a path into the compatible one 

## `Cstr / Cstring`

to make inter-compatibility between rust and c a conversion must be done for the string:
- C: 8 bit for character
- Rust: UTF-8 size for every character

## Function
Define the main behavior of the program. If return value differs from `()` the list of return values must be specified with the `-> (list_of_values)` (e.g. `-> (i32)` define the return type to be a signed 32 bit integer, if a single value is returned the parenthesis can be omitted) 
The last statement if defined without the `;` is called an expression and is interpreted as the return statement

## Nested Loop
literally defined with the keyword `loop`, with an optional label (e.g. `'label: loop{}`), so that the keyword `continue` and `break` can reference the label to break an outer loop from an inner one 

---
# Possession
#2024-03-25


In Rust every values introduced into the program is owned by ***one*** variable at a time. 
The compiler, alongside the other check, uses the **borrow checker**, verifying the ownership of a value in every instant of the program. Every violation will throw a compile error
- Having the ownership of a variable means being responsible for his **release** and happens when:
	1. The variable that have ownership *exit* from his syntactic *scope* 
	2. When a new value is assigned to the variable

## Possession and Release

```rust
fn main() { 
	// create a vec with dimension 4
	let mut v = Vec::with_capacity(4); 
				// v possiede il vec 
				
	for i in 1..=5 { 
		// the push will add value and will increase it's value
		// by a factor of two in the last iteration 
		// (5 elemnts inside of it)
		v.push(i); 
	} 
	// capacity at this point 8 
	// (from 4 in the last iteration it was doubled)
	
	println!(“{:?}”,v); 
} // here the vec is dropped and the heap is freed up
```

## Movement
When a variable is initialized take ownership of its relative value. If from a (mutable) variable is assigned a new value, the previous one is released/dropped from memory and the variable take ownership of the new value.
When a variable is assigned to a new variable or passed as arguments of a function its content is ***moved*** into the destination.
- The **original variable** doesn't have ownership anymore, instead this is moved to the **destination variable**. The destination will contains a copy bit by bit of the original value.
- The original variable **remain allocated** until it get out of scope but:
	- Eventual ***read access*** to this variable will throw a **compile error**
	- Eventual ***write access*** will **succeed** and makes the variable **readable again**

```rust
let mut s1 = “hello”.to_string(); // String does not implement the copy trait, and so it could only be moved

println!(“s1: {}”, s1); 

let s2 = s1; //s1 value moved to s1, now s1 no more accessible

println!(“s2: {}”, s2); // "s2: hello" s1 still have this value but cannot access it

// COMPILE ERROR
println!(“s1.to_uppercase(): {}”, s1.to_uppercase()); 
// value borrowed after move      ^^^^^^^^^^^^^^^^^

// But if s1 recieve a write access
s1 = "world".to_string(); // s1 now accessible in read mode
// NO ERROR!
println!(“s1.to_uppercase(): {}”, s1.to_uppercase());
```
![[Pasted image 20240325132653.png]]
`s1` and `s2` have the same value that point to the same string into the heap, but since there could be ***only one*** owner of a value at the time, the last one (`s2`) is the only one that can access it.

### Movement == consumption of variable

```rust
fn main() { 
	let s = String::from("hello"); 
	
	// s moved to takes_ownership();
	takes_ownership(s); 
	
	// COMPILE ERROR
	println!("{}", s);
	// s NO MORE ACCESSIBLE!
} 

// s value goes here and no more accessible from the main fn!
fn takes_ownership(some_string: String) { 
	let s = some_string.to_uppercase(); 
	println!("{}", s); 
}
```

## Copy
Some types, from which the numeric ones, are defined as copyable, in particular:
> Implements the trait `Copy`

When a value is assigned to a new variable or used as argument of a function the original value keeps the read operation

- Simple types and their combination (number -> tuple/array of number) are copyable
	- The same apply to reference of **not mutable** value
	- Mutable value **cannot** be copied!

From the point of view of the generated code it doesn't change anything from copy and movement!
- Assignment instruction or passing to a function as argument will create a duplication of the original value
- Simply, in case of copy, the **borrow checker** does not limit further read access

## Copy vs Movement
It's very important to understand the difference between copy and movement as the wrong usage can cause compiler error.

![[Pasted image 20240325134424.png]]

```rust
let x1 = 42; // initialize x1 as integer 42
let y1 = Box::new(84); // initialize y1 as a box
	{ 
		// initialize z as tuple from the two variable:
		// x1 is copied
		// y1 is moved!
		let z = (x1, y1);  
		println!("z: {:?}", z); 
	} 

// no problem
let x2 = x1; 
println!("x2: {}", x2); 

// y1 cannot be access in read mode! It's value was moved in the scope above into the z variable! COMPILE ERROR
let y2 = y1; 
println!("y2: {}", y2);
```

So how can use the same value and resolve this compile error! ***CLONE***
## Clone
```rust
let mut s1 = "hi".to_string(); 

// clone all reference from the original value and assign it to s2
let s2 = s1.clone(); 
// s1 can be accessed without problem, the two variable are not correlated
s1.push('!'); 

println!("s1: {}", s1); //hi! 
println!("s2: {}", s2); //hi
```

![[Pasted image 20240325135044.png]]

## Differences with C/C++
Instead of what we're used to think when programming in older languages like `c/c++`, in rust the behavior of the compiler against the assignment and other operation to a value is the **movement** 

- `C`: only paradigm is the copy
- `C++`: movement is possible but there is no equivalence of **borrow checking** and is the programmer that need to be certain to not access the moved variable, release the object and others.

## Reference
To resolve difference types of problem #Rust introduce various pointers with main focus to explicitly declare responsibility of who can manage it

A **reference** is a pointer read-only to a block of memory owned by another variable
```rust
// point possiede il valore 
let point = (1.0, 0.0); 

// reference PUÒ accedere al valore in lettura 
// finché esiste point 
let reference = &point; 

// no error!
println!(“({}, {})”, reference.0, reference.1);
```

### Reference and Borrows
A reference *borrows* the memory address from which the the value exist. While the reference is accessible the value **cannot** be modified:
- By reference (write-only)
- By variable (the owner of the value)
Cannot create data-hazard!
### Mutable Reference
From a variable that own a value is possible to extract **only one** mutable reference at the time. 
This is done using the syntax
```rust
// A mutable reference can be done only to a mutable variable
let mut v = 42;
let r = &mut v;
```

While a mutable reference exist:
- There could not be simple reference (read-only)
- The owner variable cannot be moved nor modified
![[Pasted image 20240325135829.png]]

And so the previous code that will have a throw a `compile error` from the `borrow checker` because of the move
```rust
fn main() { 
	let s = String::from("hello"); 
	
	// s moved to takes_ownership();
	takes_ownership(s); 
	
	// COMPILE ERROR
	println!("{}", s);
	// s NO MORE ACCESSIBLE!
} 

// s value goes here and no more accessible from the main fn!
fn takes_ownership(some_string: String) { 
	let s = some_string.to_uppercase(); 
	println!("{}", s); 
}
```

Can be easily fixed by using reference:
```rust
fn main() { 
	let s = String::from("hello"); 
	
	// s reference passed as arguments to takes_ownership();
	takes_ownership(&s); 
	
	println!("{}", s); // print "hello"
} 

// need to change arguments type to a ref
fn takes_ownership(some_string: &str){ 
	let s = some_string.to_uppercase(); 
	println!("{}", s); // print "HELLO"
}

// OTHER USAGE OF REFERENCE

fn main() { 
	let s = String::from("hello"); 
	
	// can modify the original variable using a reference as the argument of the function
	s = takes_ownership(&s); 
	
	println!("{}", s); // print "HELLO"
} 

// need to change arguments type to a ref
fn takes_ownership(some_string: &str) -> String{ 
	let s = some_string.to_uppercase(); 
	println!("{}", s); // print "HELLO"
	s // return "HELLO"
}
```

Let's analyze some other cases
```rust
fn main() { 
	let mut b = Box::new(84); 
	let r = & mut b; 
	*r = Box::new(100); 
	// immutable borrow occurs
	println!("{:?}", b); 
	// mutable borrow used later
	println!("{:?}", r); 
	// COMPILE ERROR
}

// FIXED

fn main() { 
	let mut b = Box::new(84); 
	let r = & mut b; 
	*r = Box::new(100); 
	println!("{:?}", r); 
	// To this point r is not used anymore, then b can be accessed in read mode
	println!("{:?}", b); 

}
```

The following code seems to cause the same problem as the code above
```rust
use rand::Rng; 

fn main() { 
	let mut b = Box::new(84); 
	let r = & mut b; 
	*r = Box::new(100); 
	
	let mut rng = rand::thread_rng(); 
	let n = rng.gen_range(0..10); 
	
	if n > 5 { 
		println!("{:?}", b); 
	} 
	else { 
		println!("{:?}", r); 
	}
}
```
In reality this will compile without any problems! The reason is that there is no way to both `b` and `r` to be accessed one after the other. The compiler will check every possible scenario to understand if a problem may occurs.


### Memory arrangement
References are implemented based on type of pointers:
1. **Simply pointers**: If compiler knows dimension of pointed value
2. Pointer + dimension (**fat pointer**): if the dimension of pointed value is known at execution time
3. **Double pointer**: if dimension of pointed value i known only by the set of traits that implements

![[Pasted image 20240325144622.png]]

## Time to live
The **borrow checker** guarantees all access to a reference to happens during an interval of time in where the value exist.
All the sets of rows where there is an access to the reference make up its *lifetime*.
Although in many situations it can be omitted, the time of life of a reference can be explicit stated in the signature of the type: `&'a TypeName` where `a` is the signature and the `'` (tick) qualify it to the time to live. This notation it useful when there is some necessity to make some constrains around the time to live of two or more reference.
When a reference is valid for the entire duration of the program, it is noted with the statement `&'static TypeName`
- An example is the string that has the type `&'static str` as it's sequence of char are allocated from the compiler in the constant section and never to be released until the end of the program

To be legit, time of life of the reference must be inside the time of life of the pointed value.

## Possession - In a few word
- Each value has a single possessor (variable or field of a structure) 
	- The value is released (drop) when the possessor leaves their scope or when the possessor is assigned a new value 
- There can exist, at most, a single mutable reference to a given value 
- Or, there can be many immutable references to the same value 
	- But as long as there's at least one, the value can't be changed 
- All references must have a shorter lifespan than the value they refer to

## Slice
A slice is a **view** of a contiguous section of elements, from which the length **is not** known at compile time, only at execution time
> Internally a slice is just a tuple of two elements:  
1. First value of the sequence
2. Numbers of elements in the sequence

A slice of elements `T` has type `[T]`

The slice has no ownership of the value it refers to:
- they remain owned by the original variable from which the slice derives
- All the values are guarantee to be initialized ad all the access are verified at execution time.

```rust
// a è un array di 5 interi 
let a: [i32; 5] = [1, 2, 3, 4, 5]; 

// s è una slice formata da 2 elementi [2,3] 
let s = &a[1..3]; 

// two contiene il valore 2
let two = s[0]; 
```

Other

## Possession Advantages
Rust, although without garbage collector, still offers many guarantees around correctness of access and release of memory resources.
- No null reference (no risk of deference)
- Borrow checker
	- No risk of segmentation fault or illegal access
	- no buffer overflow or underflow
- Rust iterator never exceed their limits

All variables are immutable by default, and thus need an explicit declaration to make it mutable.
Possession model not only guarantee memory safety, but also resource management inside of a variable

The **absence of garbage collector**, although may create some fear, it will ensure deterministic behavior (in particular no need to suspend a function to organize the memory)

---
# Composite Type
In `C/C++` the keyword `struct` let create a new type that will contains a group of fields with public accessibility.
Or others composite type like:
- `enum`: define a set of constant that are associated by compiler or by the programmer with distinct `integer` values
- `union`: make possible to use the same block of memory to represent values of different types

```c
union sign 
/* A definition and a declaration */ 
{ 
	int svar; 
	unsigned uvar; 
} number;

enum DAY 
/* Defines an enumeration type */ 
{ 
sunday = 0,  /* Names day and declares a */ 
monday,      /* variable named workday with */ 
tuesday,     /* that type */ 
wednesday,   /* wednesday is associated with 3 */ 
thursday, 
friday, 
saturday     /* saturday is associated with 6 */ 
} workday;
```

##### Particularly in C++
It is possible to associate method at instance/type level (static) or, moreover, introducing the new keyword `class`
It is similar to the `struct` but let decide the public access to a subset of it's contents, new keyword for different situation `public, protected, private`
```c++
class Foo { 
public: 
	   // Methods and members here are publicly visible 
	   double calculateResult(); 
	   
protected: 
	   // Elements here are only visible 
	   // to this class and to its subclasses 
	   double doOperation(double lhs, double rhs); 
	   
private: 
	   // Elements here are only visible to ourselves 
	   bool debug_; 
};
```

## Struct in Rust
Usually there are some needs to maintains joined some heterogeneous information, as this is possible to use `Tuple<T>` this has the tendency to hide the semantics of the value: a tuple with two integer may contains a fraction or some coordinates.
When there is the need to create a peculiar semantic of a data structure, it's possible to use the `struct`
It is a construct that makes possible to represent some data sequentially in a block of memory, where the name and type are defined by the programmer
```rust
struct Player { 
	name: String, // nickname 
	health: i32,  // stato di salute (in punti vita) 
	level: u8,    // livello corrente 
}

fn main(){
	let name = “Mario”.to_string();
	let health = 25;
	let level = 1;
	
	
	// the Struct field may be defined by name 
	let mut s = Player { 
		name: “Mario”.to_string(), health: 25, level: 1 
	};
	
	
	// by position {name: name, health: health, level: level}
	let p = Player { name, health, level }; 
	
	// or by using another instance of the same struct
	let s1 = Player {name: “Paolo”.to_string(), .. s};
	// where the omitted field are obtained by the other struct
}
```

By convention the struct uses the first letter uppercase with the rest of the name in **CamelCase**

To access the single field, the dot notation is used
```rust
println!(“Player {} has health {}”, s.name, s.health );
s.level += 1; //remember s was defined as mutable
```

Although the reason of the `Struct` was to make tuple more understandable to the programmer, the struct can be defined similar to the tuple (useful when need a named Tuple)

```rust
struct Playground ( String, u32, u32 ); 
struct Empty; // non viene allocata memoria per questo tipo di valore 

let mut f = Playground( “football”.to_string(), 90, 45 ); 
let e = Empty;


struct Point (i32, i32, i32); 
fn main() { 
	let p1 = Point(0, 0, 0);   // named Tuple 
	let p2 = (0, 0, 0);        // Tuple without 
	println!("{:?} \n{:?}", p1, p2); 
	println!("{} {} {}", p1.0, p1.1, p1.2); 
}
```

## Module
In rust the class does not exist, and by default the privacy is limited to the module

```rust
mod Module1 { 
struct Test{ 
		a: i32, 
		b: bool 
	} 
} 

fn main() { 
	// COMPILE ERROR Test inside of Module1: cannot access it!
	let t = Test { a: 12, b: false}; 
	println!("Hello, world!"); 
} 
```

To make module usable and accessible it's mandatory to define the public field of it
```rust
mod Module1 { 

pub struct Test{ 
	a: i32, // <-ATTENTION a not public!
	pub b: bool
} 
fn f () { 
	let t = Test { a: 1, b: false}; 
	println!("Hello, main !"); 
} 
} 

use Module1::Test; 
fn main() { 
	let t = Test { a: 12, b: false};
//COMPILE ERROR   ^^^^^^ 'a' WAS NOT DEFINED PUBLIC! b is fine
	println!("Hello, world!"); 
}
```

### Visibility
This makes the module capable of implements encapsulation mechanism (*information hiding*). 
- To be effective need to associate a set of **behavior** (**methods**) to the struct
- Differently of how in other languages is done, in which the struct and methods are defined contextually in a single block (*class*), in #Rust definition of methods relative to the **struct** happens separately, in a block of type `impl` (implements)

### Methods
To add a method to any type (a struct, a primitive type, etc..), it's as easy as using the keyword `impl` (*implements*) followed by the `type_name` and it's methods and with the keyword `self` , `&self` or `&mut self` as first parameter.
```rust
// First define the data field
struct Something { 
	i: i32, 
	s: String 
} 

// Then define the methods implemented by a struct 
// with its parameters
impl Something { 
	// pass the self as arguments and cosume it! DON'T DO THIS
	fn take_ownership(self) {...} 
	// pass a reference to self 
	fn process(&self) {...} 
	// pass a mutable reference to self 
	fn increment(&mut self) {...} 
}
```

`self` is the receiver, keyword similar to `this` from other programming language but there are some limitation.

Rust is **not** a traditional object oriented programming language, thus don't have the class concept. Although structs may seems similar to classes from other programming language, the similarities are limited:
- Struct **don't have** hereditary
- Concept of methods apply to every types (including the primitives)


```rust
struct Point { 
	x: i32, 
	y: i32, 
} 

impl Point { 
	
	fn mirror(self) -> Self { 
		// consume the self to invert it's point
		Self{ x: self.y, y: self.x } 
		// Self: type Point
		// self: type Point consumed into the mirror
	} 
	
	fn length(&self) -> f64 { 
		let a = self.x*self.x + self.y*self.y;
		let res = (a as f64).sqrt(); 
		res 
	} 
	
	fn scale(&mut self, s: i32) { 
		self.x *= s;
		self.y *= s; 
	} 
}

/*
It would be possible to implement in different position without overwriting them!

impl Point { 
	
	fn mirror(self) -> Self { 
		Self{ x: self.y, y: self.x } 
	} 
	
	fn length(&self) -> f64 { 
		let a = self.x*self.x + self.y*self.y;
		let res = (a as f64).sqrt(); 
		res 
	} 
}

impl Point { 	
	fn scale(&mut self, s: i32) { 
		self.x *= s;
		self.y *= s; 
	} 
}

*/
```

### Constructor
In Rust there is no concept of constructor. As rust does not support function overloading, every constructor must have a different name, as a nice rule
```rust
// constructor with no arguments
pub fn new()-> Self{...}

// constructor with some 
pub fn with_details(...) -> Self {...}
```

An example is
```rust
#[derive(Debug)] 
struct Player { 
	name: String, // nickname 
	health: i32, // stato di salute (in punti vita) 
	level: u8, // livello corrente 
} 

impl Player { 
	// constructor to create a specific player using default setting and modifying name (a string)
	fn with_name(s: String) -> Self { 
		Self {name: s, health: 25, level: 1} 
	} 
} 

fn main() { 
	let p1 = Player::with_name("Mario".to_string()); 
	println!("{:?}", p1); 
}
```

### Destructor
In `C++` the `~` keyword is used to define how the object is destructed and is automatically called when the the object exit its scope 

In rust it must implement the `drop` trait

```rust
pub struct Shape { 
	pub position: (f64, f64), 
	pub size: (f64, f64), 
	pub type: String 
}

impl Drop for Shape { 
	// ref mut because it has to change it's value (ATTENTION TO THIS)
	fn drop(&mut self) { 
		println!("Dropping shape!"); 
	} 
}
```

The trait `Drop` is mutually exclusive with trait `Copy`, if a type implement one **cannot** implement the other

### Static Method
In rust instead of using the keyword `static` to implement a static method the only need is to not define the `self` nor its derivative as first argument, they're also called associated functions, and are defined to the type instead of the *instance*

Those methods are called with the sequence `<Type>::static_method()`:
```rust
let s = String::from("Hello world!");
let mut var = Vec::new();
```

If you pay attention is the same concept as the constructor of the player above where a type `Player` is defined using a string!

## Enum
In rust it is possible to introduce #Enum type composite by a simple scalar value. It is also possible to attach a method to an Enum:
```rust
// usual case
enum HttpResponse { 
	Ok = 200, 
	NotFound = 404, 
	InternalError = 500 
}

// enum with method
enum HttpResponse { 
	Ok, 
	NotFound(String), 
	InternalError { 
		desc: String, 
		data: Vec<u8> 
	}, 
}
```

Enum is defined as `sum type`:
- Set of values that can contains: is the union of the values of the singular alternative
Struct is `product type`:
- Set of values that it can contains is the cartesian of the sets in respect of the singular field

In memory #Enum take integer of 1 byte (the tag) plus the space to take the max size variable from its possible ones

| Memory allocation                    | Code                                                                      |
| ------------------------------------ | ------------------------------------------------------------------------- |
| ![[Pasted image 20240326123820.png]] | <br>enum Shape { <br>Square(u32), <br>Point(u8, u8), <br>Empty, <br>}<br> |
So in the example above the first byte is used to define which of the three value is currently assigned to the enum. Then there is the space to contains at least the biggest value. As `Square` uses 4 byte plus the 1 byte of the Tag the memory would occupy 5 byte, thus 8 byte are reserved in this situation.

```rust
enum Operation {
	Add,
	Subtract
}

impl Operation {
	fn run(&self, x: i32, y: i32) -> i32 {
		// Enum well fitted for match case 
		// all it's value will be enough 
		// to satisfy rust completness compiler
		match self {
			Self::Add => x + y,
			Self::Subtract => x - y,
		}
	}
}

fn main(){
	let op1 = Operation::Add;
	let op2 = Operation::Subtract;
	
	// use enum add operation
	let res = op1.run(5,7);
	println!("{}", res); // 12
	
	let res = op2.run(5,7);
	println!("{}", res); // -2
}
```

Another way to pattern match is the construct
```rust
if let <pattern> = <value> … 
while let <pattern> = <value>


fn process(shape: Shape) { 
	// print only if shape is a Square
	if let Square { s } = shape { 
		println!("Square side {}", s); 
	} 
}
```

### `Option<T>`
Type of Enum defined in the *standard library* that let make possible represent the case in which a value can be present or absent

`Option<T>` contains two possible values:
- `Some(T)`: value is present and contains a value
- `None`: value absent


```rust
fn divide(numerator: f64, denominator: f64) -> Option<f64> { 
	if denominator == 0.0 { 
		None 
	} else { 
		Some(numerator / denominator) 
	} 
} 

fn raddoppia (x: Option<f64> ) -> Option<f64> { 
	match x { 
		None => None, 
		Some(i) => Some(i*2.0), 
	} 
} 

fn main() { 
	let result = divide(820.0, 15.2); 
	let numero = raddoppia (result); 
	match numero { 
		Some(numero) => println!("Risultato della divisione moltiplicato per 2 vale {}", numero), 
		None => println!("Non è possibile eseguire la divisione."), 
	} 
}
```

### `Result<T, E>`
- `Ok(T)`: if operation successful return value `T`
- `Err(E)`: if failed return type `E`

Both letters are just convention to differentiate between the different types.

Dividing two number (Need to check for division by zero)
```rust
// the Result<T, E> is the enum returned by the function
fn divide(x: f64, y: f64) -> Result<f64, &'static str> { 
	if y == 0.0 { 
		// Returning an error if division by zero is attempted 
		Err("Division by zero!") 
	} else { 
		// Returning the result of the division 
		Ok(x / y) 
	} 
} 
fn main() { 
	// Attempting division 
	let dividend = 10.0; 
	let divisor = 2.0; 
	// Handling the result 
	match divide(dividend, divisor) { 
		Ok(result) => println!("Result: {}", result), 
		Err(err) => println!("Error: {}", err), 
	} 
}
```

Conversion String to Integer
```rust
fn string_to_int(s: &str) -> Result<i32, &'static str> { 
	// parsing the string into i32, if ok return n, else reuturn error string
	match s.parse::<i32>() { 
		Ok(n) => Ok(n), 
		Err(_) => Err("Impossibile convertire la stringa in un numero intero."), 
	} 
} 

fn main() { 
	match string_to_int("mamma") { 
		Ok(num) => println!("Il numero convertito è: {}", num), 
		Err(message) => println!("{}", message), 
	} 
}
```


# Polymorphism 
#2024-04-09 
> #Polymorphism derive from the Greek word $πολύς$ (multiple) and $μορφή$ (forms)

The concept is use a generic template from which derive a specialization to avoid repetition.
- Useful to avoid bug from `ctrl+c, ctrl+v`
and helps make specific implementation more easy.
- Problematic in some cases. Use with brain!


## #Cpp implementation

Define a generic method es. `getValue()` and then implement the specific way this function work for each case:
- First class may return a String with two value `es Name, Surname` and the second the list of people attending a course `es []`

But how it works? #vtable

need to check for every class it's implementation and thus adding different memory access and inefficiency other then complexities. 

Multiple inheritance is possible (can be a bad thing! #UnpredictableBehaviorToBeExpected)

## #Rust implementation
#Traits are a collection of methods that define a behavior that a **type** *can* **implement**
## Traits & Generics
Uses of traits implements <mark style="background: #FF5582A6;">(in most cases)</mark> no overhead (no penalties for host pointer in the #vtable!)
> only exception when using a double pointer `&dyn TraitName` (reference to a traits that is an abstract thing)
> because we have **no dimension at compile time**!

A type that implement the trait `std::clone::Clone` can make copies of it's own value

### Define a Trait
To define a trait use the keyword `trait`:
```rust
trait SomeTrait { fn someOperation(&mut self) -> SomeResult; … }
```

To implement a trait use the keyword `impl`:
```rust
impl SomeTrait for SomeType { … }
```

To import a trait use the keyword `use`:
```rust
use SomeNamespace::SomeTrait;
```

Some traits don't need to be explicit wrote (es. `Clone` or  `Iter`), this applies to traits from the standard library (`prelude`) that are automatically imported for every crate. 

The implementation can be done both where the trait is defined or where the type is define, <mark style="background: #FF5582A6;">not</mark> in another file!

Definition of a trait
```rust
trait T1 { 
	fn returns_num() -> i32; 
//ritorna un numero 
	fn returns_self() -> Self; //restituisce un’istanza del tipo che lo implementa 
}
```

How implementation make coding more generic and let's create inheritance 
```rust
struct SomeType; 
impl T1 for SomeType { 
	fn returns_num() -> i32 { 1 } 
	fn returns_self() -> Self {SomeType} 
}
```

```rust
struct OtherType; 

impl T1 for OtherType { 
	fn returns_num() -> i32 { 2 } 
	fn returns_self() -> Self {OtherType} 
}
```


```rust
trait T2 { 
	fn takes_self(self); 
	fn takes_immut_self(&self); 
	fn takes_mut_self(&mut self); 
}
/***********************/
/*                     */
/*     EQUIVALENT      */
/*                     */
/***********************/
trait T2 { 
	fn takes_self(self: Self); 
	fn takes_immut_self(self: &Self); 
	fn takes_mut_self(self: &mut Self); 
}
```

Es. code
The struct `Persona` and `Azienda` implement the trait `Salutabile` and define their custom `saluta(&self)` method with their specific need

Similarly for a geometric problem
The struct `Circle` and `Rectangle` implement the trait `HasArea` and define their custom `get_area(&self)` method with their specific need
More specific, the variable self when the `circle` is defined from the struct `Circle` reference the variable `circle`! So you can have multiple variable of type `Circle` with their own values.

> Not all traits method must have the `self, &self, &mut,` parameter! 

The `self` parameter It is used to make a method that reference it's value, but it's not always needed, so you can avoid using it if not needed:
```rust
trait Default { 
	fn default() -> Self 
	// Self return the type that has the traits implemented, it's not a reference of the value that invoke the method
	// circle   !=   Circle
	// variable !=   Type
}

fn main() { 
	let zero: i32 = Default::default(); 
	let zero_again = i32::default(); 
}
```

### Subtrait and Supertrait
It is possible to make multiple inheritance

```rust
trait Supertrait { 
	fn Sup(&self) {println!(“In super”);} 
	fn g(&self) {} 
}

trait Subtrait: Supertrait { 
	fn Sub(&self) {println!(“In sub”);} 
	fn h(&self) {} 
}

/****************/

struct SomeType; 

impl Supertrait for SomeType {} 
impl Subtrait for SomeType {} 

fn main() { 
	let s = SomeType; 
	s.Sup(); 
	s.Sub(); 
}
```

When the trait `Subtrait` is defined, the usage of `: Supertrait` make it depend on the other trait when a type try to implement it

### Object-trait
Reference to traits are called `object-trait` and are implemented using `fat pointer` or `double pointer`

```rust
&dyn TraitName
```

![[Pasted image 20240409125642.png]]

Es.

```rust
trait Shape { 
	fn area(&self) -> f64; 
} 
trait Drawable: Shape { 
	fn draw(&self);
}

struct Square { side: f64, } 
impl Shape for Square { 
	fn area(&self) -> f64 { self.side * self.side } 
}

impl Drawable for Square { 
	fn draw(&self) { 
		println!("Drawing a square with side {}.", self.side);
	}
}
// pass a type that implement the Drawable trait as arguments
fn draw_shape(shape: &dyn Drawable) { 
	shape.draw(); 
	println!("Area: {}", shape.area()); 
} 

fn main() { 
	let square = Square { side: 2.0}; 
	draw_shape(&square); 
}
```


## std::Library
Rust define a set of basic traits that the eventual implementation enables syntactic shortcuts with operator presents in the language
I.e. operators `==` or `!=` are available when the type implement the traits **Eq** or **PartialEq**
Similarly `< > <= >=` are defined from **Ord** and **PartialOrd**
Finally also the operators `+ - * / % & ^ << >>` are defined respectively from the traits:  **Add**, **Sub**, **Mul**, **Div**, **Rem**, **BitAnd**, **BitXor**, **Shl**, **Shr**

This means that all those syntax can change how you operate with special struct by implementing those traits:
```rust
#[derive(Debug)] 
struct Point { 
	x: i32, 
	y: i32, 
} 

impl PartialEq for Point { 
// The PartialEQ make you implement the function eq
	fn eq(&self, other: &Self) -> bool { 
		// redefine the double equal operator to define a bool
		// output checking if both x and y parameters are equals
		self.x == other.x && self.y == other.y 
	} 
} 

fn main() { 
	let point1 = Point { x: 1, y: 2 }; 
	let point2 = Point { x: 1, y: 2 }; 
	let point3 = Point { x: 3, y: 4 }; 
	println!("point1 == point2: {}", point1 == point2); // true 
	println!("point1 == point3: {}", point1 == point3); // false 
}
```

Traits `PartialEq` and `Eq` are similar, the first one is the most general one, and require two property: transitivity and symmetry. `Eq`, sub trait of `PartialEq`, require, in addition of the other two properties, also the reflective property.
Floating point (`f32, f64`) does **NOT** implement this trait as the `NaN != NaN`.
`Eq` has no more method than `PartialEq`, it is used only to specify that the equal method has the reflective property.
The method `ne()` is usually defined as the opposite result of `eq()`

## Automatically derive methods

Using the `derive` keyword, it is possible to automatically implements some methods
```rust
#[derive(PartialEq)] struct Foo<T> { 
	a: i32, 
	b: T, 
}
```

The previous code will result in the following structure
```rust
impl<T: PartialEq> PartialEq for Foo<T> { 
	fn eq(&self, other: &Foo<T>) -> bool { 
		self.a == other.a && self.b == other.b 
	} 
	
	fn ne(&self, other: &Foo<T>) -> bool { 
		self.a != other.a || self.b != other.b 
	} 
}
```

Thus, the Point example done before where we manually implemented the eq method can be simplified like the following
```rust
#[derive(Debug, PartialEq, Eq)] 
struct Point { 
	x: i32,
	y: i32, 
} 

fn main() { 
	let point1 = Point { x: 1, y: 2 }; 
	let point2 = Point { x: 1, y: 2 }; 
	let point3 = Point { x: 3, y: 4 }; 
	// Two points are equal if all it's field are equal
	println!("point1 == point2: {}", point1 == point2); // true 
	println!("point1 == point3: {}", point1 == point3); // false 
}
```

Another example of trait that can be implemented:
```rust
std::cmp::Ordering;

#[derive(Debug, Eq, PartialEq, PartialOrd)] 
struct Person { 
	name: String, 
	age: i32, 
} 
impl Ord for Person { 
	fn cmp(&self, other: &Self) -> Ordering { 
		other.age.cmp(&self.age) 
	} 
} 

fn main() { 
	let john = Person { name: String::from("John"), age: 25 }; 
	let emma = Person { name: String::from("Emma"), age: 23 }; 
	let peter = Person { name: String::from("Peter"), age: 25 }; 

	// Stampa: John is Greater, compared to Emma. 
	println!("John is {:?}, compared to Emma.", john.cmp(&emma)); 
	// Stampa: John is Equal, compared to Peter. }
	println!("John is {:?}, compared to Peter.", john.cmp(&peter)); 
}
```


## Resource Release
Trait `Drop` lets define the method `drop(&mut self)` equivalent of the destructor of C++
The rust compiler guarantees a call to that method when the variable get out of its scope

This trait is mutually exclusive with the trait `Copy` (to avoid a double release)

## Index data structure
It's possible to use syntactically a data structure as if it was an array, enabling it the notation `t[i]` when implementing the traits `Index` and `IndexMut`
```rust
trait Index<Idx> { 
	type Output: ?Sized; 
	fn index(&self, index: Idx) -> &Self::Output; 
} 

trait IndexMut<Idx>: Index<Idx> { 
	fn index_mut(&mut self, index: Idx) -> &mut Self::Output; 
}
```

## Deref
When using pointer you have a variable with a memory address, if you need the actual value, you need to deference that pointer
```rust
// the variable will have the memory address where the integer is stored
let string: &str = "My String";

// to get the actual string use the keyword *

let actual_string = *string; //not sure if valid in this context, take it as an example
```


When dealing with struct, the trait `Deref` let's you specify what value is taken when deferencing
```rust
use std::ops::Deref; 
struct MyData {value: i32,} 

struct Container {data: Box<MyData>} 
impl Deref for Container { 
	type Target = MyData; 
	
	fn deref(&self) -> &Self::Target { 
		&self.data 
	} 
} 

fn main() { 
	let my_data = MyData { value: 42 }; 
	let container = Container { data: Box::new(my_data) }; 
	// dereferenziazione automatica 
	// per accedere al valore all'interno di Container 
	println!("Value inside Container: {}", container.value); 
}
```

## Type Conversion - From, Into
The traits `From` and `Into` lets you make conversion of types. Take ownership of value and convert it, and return the ownership of the new value to the caller:

```rust
trait From<T>: Sized { 
	// !ATTENTION no reference of other! Ownership required!
	fn from(other: T) -> Self; 
} 

trait Into<T>: Sized { 
	fn into(self) -> T; 
}
```

and so applied to an example Point struct to create a Point from an array of two elements
```rust
struct Point { 
	x: i32, 
	y: i32, 
} 

impl From<(i32, i32)> for Point { 
	fn from((x, y): (i32, i32)) -> Self { 
		Point { x, y } 
		} 
	} 

impl From<[i32; 2]> for Point { 
	fn from([x, y]: [i32; 2]) -> Self { 
	Point { x, y } 
	} 
}

fn main() { 
	// from 
	let p1 = Point::from((3, 1)); 
	let p2 = Point::from([5, 2]); 
	
	// into 
	let p3: Point = (1, 3).into(); 
	let p4: Point = [4, 0].into(); 
	
	// ERRORE! non vale la simmetria 
	let a1 = <[i32; 2]>::from(point); 
	let a2: [i32; 2] = point.into(); 
	let t1 = <(i32, i32)>::from(point); 
	let t2: (i32, i32) = point.into(); 
}
```

or can be used to convert to different metrics system
```rust
struct Centimeters(f64); 

// Definiamo una struttura Inches 
struct Inches(f64); 

// Implementiamo il tratto From 
// per convertire da Inches a Centimeters 
impl From<Inches> for Centimeters { 
	fn from(inches: Inches) -> Self { 
		Centimeters(inches.0 * 2.54) 
	} 
} 
fn main() { 
	let inches = Inches(10.0); 
	let centimeters: Centimeters = Centimeters::from(inches); 
	println!("10 pollici sono equivalenti a {:.2} centimetri.", centimeters.0); 
}
```

Similarly can be done for other context

```rust
struct Person { 
name: String, age: u32, 
} 
struct Employee { name: String, position: String, }

impl From<Person> for Employee { 
	fn from(person: Person) -> Self { 
		Employee { 
			name: person.name, 
			position: String::from("Sviluppatore"), 
		} 
	} 
} 

fn main() { 
	let person = Person { 
		name: String::from("Mario Rossi"), 
		age: 30, 
	}; 
	// !ATTENTION: person is consumed!
	let employee: Employee = Employee::from(person); 
	
	println!("Nome: {}", employee.name); 
	println!("Posizione: {}", employee.position); 
	
	// ERROR person was consumed when converted into employee!
	println!("Nome: {}", person.name);
}
```

### `TryFrom`, `TryInto`, `FromStr`
When conversion may fails the trait `TryFrom` or `TryInto` are used. There is a trait specifically to convert from a string: `FromStr`
```rust
pub trait TryFrom<T>: Sized { 
	type Error; 
	fn try_from(value: T) -> Result<Self, Self::Error>; 
} 

pub trait TryInto<T>: Sized { 
	type Error; 
	fn try_into(self) -> Result<T, Self::Error>; 
}

pub trait FromStr { 
	type Err; 
	fn from_str(s: &str) -> Result<Self, Self::Err>; 
}
// All of this traits return Result! -> <Ok_value, Error>
```

# Generics
Using type lets create robust code.
Sometimes to augment or to fix some issue, is needed to implement new features and thus maybe is needed to replicating a big amount of code. This may cause some issue because you need to implement those modification to all the implementation whenever personal types are used.
To avoid this, generics type can be used.

```rust
fn max<T>( t1: T, t2: T) -> T where T: Ord { 
	return 
		if t1 < t2 { t2 } 
		else { t1 } 
}
```
Note that in this case the generic type `T` must implement the trait `Ord` to make possible the implementation of the function where two variable are compared

Although in #Cpp the functions may operate with every type and is the programmer job to avoid using wrong type,
In #Rust the implementations are much more strict. (as the example above may suggest, if the function implement some type of comparison, the type passed must implement the trait that let the operation used work).

This lets the rust compiler create efficient (in term of speed with more memory used) code that will generate specific function ad hoc for the specific type every time is invoked
```rust
// generic function
fn max<T>( t1: T, t2: T) -> T where T: Ord { 
	return 
		if t1 < t2 { t2 } 
		else { t1 } 
}

let max_int = max<i32>(1, 2);

let max_float = max<f32>(1.0, 2.0);
```
This example will generate two implementation of the generic max function, one for the integer and one for the floating type that will greatly speed up the execution as there will be not generic function to implement at run-time, at the cost of more memory space used (it will be equivalent of manually implement the specific function for every type).
This will have a great advantages of having a very fast code with minimal maintenance as it will be regenerated for every type at compile time, and if some modification are needed, the only modified function will be the generics that will be the "generator" of the specific function for the compiler to make.

## Bound
In #rust there are two way to specify type constraints
```rust
<T: SomeTrait>
<T> … where T: SomeTrait
```
With a code example
```rust
// M must implement both Mapper and Serialize
// R must implement both Reducer and Serialize
fn run_query<M: Mapper + Serialize, R: Reducer + Serialize>( 
		data: &DataSet, map: M, reduce: R) -> Results { 
		// function implementation 
}

fn run_query<M,R>( data: &DataSet, map: M, reduce: R) -> Results 
	where M: Mapper + Serialize, 
		  R: Reducer + Serialize { 
	// function implementation 	
}
```

## Generic Trait
Is possible to create also a ***generic trait***. This means a trait that have a generic type in its operation
```rust
trait Confrontabile<T> { 
fn confronta(&self, other: &T) -> bool; 
} 

// Implementazione del tratto Confrontabile 
// per i tipi di dati numerici 
impl<T> Confrontabile<T> for T 
	where T: std::cmp::PartialOrd, { 
	fn confronta(&self, other: &T) -> bool { 
		self >= other 
	} 
} 

fn main() { 
	let x = 5; 
	let y = 10; 
	println!("x è maggiore o uguale a y? {}", x.confronta(&y)); 
}
```

object-trait requires uses of *double pointer* for letting the access.

```rust
fn dynamic_process(w: &mut dyn Write) { … } 
// or
fn generic_process<T>(w: &mut T) where T:Write { … }
```

# Pointer to Function
#2024-04-23
As other languages, also rust, lets the creation of pointer to function.
```rust
fn f1(i:i32, d:f64)->f64{
	return i as f64*d;
}


let ptr: fn(i32, f64)->f64;
ptr = f1; // ptr is a pointer to the function

// call the function using the pointer
let num = ptr(2, 3.14);

```
This allow the call of a function using other "name". 
There are other more advanced implementation of this functionality:
```rust
fn add(a: i32, b: i32) -> i32 {a + b}
fn subtract(a: i32, b: i32) -> i32 {a - b}
fn multiply(a: i32, b: i32) -> i32 {a * b}


fn main(){
	let operation_name = "add";
	
	let (x,y) = (10,5);
	
	let operation: fn(i32, i32) -> i32 = match opration_name{
		"add" => add,
		"subtract"=> subtract,
		"multiply"=> multiply,
		_ => panic!("Operation not supported"),
	};
	let result = operation(x,y);
	
	println!("The result of the operation {} is: {}", operation_name, result);
}
```
This will allow to call multiple function using a single "name" based on a match of a string
> possible implementation with user input that decide what operation to do. Don't need to call the different function inside the match with it's respective println! Just set the type of function of the pointer inside of it and then call it in the outside scope

Otherwise it would have gone out of scope and the print wasn't able to get the result value

# Lambda Function
- Function of superior order
- Anonymous function
- Lambda function
- Closures

> All the same thing!


## Closures

### Examples

Closure usually have the compiler figuring it out the variable types
```rust
fn main(){
	let add = |num1, num2| num1+num2;
	pritnln!("La somma è : {}", add(1,2));
}
```

But it can be explicitly typed if needed to use the function for a specific type or just for better documentation
```rust
fn main(){
	let multiply  = |x: i32, y: i32| -> i64 {
		(x*y).into()
	};
	pritnln!("{}", multiply(3, 4)); // print 12
}
```

Those function can use scoped variable
```rust
fn main(){
	let factor = 2;
	let multiply  = |n| n * factor;
	pritnln!("{}", multiply(5)); // print 10
}
```

Which are the free variables?

`factor` is a  free variable captured to make the operation inside of the function possible

A possible implementation is also for some methods that require a closure to work properly, like the method `sort_by()`

```rust
let mut numbers = vec![...]
numbers.sort_by(|a, b| a.cmp(b));
```
Imagine a `vec` of numbers, to sort it we use a closure to compare every numbers to the next one using a closure to define the behavior and define what kind of sorting algorithm we want to use (in this case the closure expect a boolean value as a return value)

The same can be applied to the `filter` method
```rust
let mut numbers = vec![...]
numbers.filter(|x| x %2 == 0);
```
In this case filtering all the even values.
It depends on the closure expected for how many parameters are needed
- `sort_by()` expect 2 parameters
- `filter()` expect a single parameter
- Other method may expect an x amount of parameters

```rust
fn main() { 
	let mut count = 0; 
	let mut increment = move || { 
		// Incrementiamo la variabile catturata 
		count += 1; 
		println!("Il conteggio è: {}", count); 
	}; 
	
	increment(); // Il conteggio è: 1
	increment(); // Il conteggio è: 2
	
	println!("Hello, {}", count); // Il conteggio è: 0
}
```
The variable count is copied and then moved inside of the closure is copied, when the referencing is finished the original value is restored thus the print will show the last referenced value `0`

```rust
fn main() { 
	let mut count = 0; 
	let mut increment = move || { 
		// Incrementiamo la variabile catturata 
		count += 1; 
		println!("Il conteggio è: {}", count); 
	}; 
	
	increment(); // Il conteggio è: 1
	// the println try to access a moved variable that cannot be accessed
	//println!("Hello, {}", count); // Il conteggio è: 0
	//^ COMPILE ERROR
	
	increment(); // Il conteggio è: 2
	
	println!("Hello, {}", count); // Il conteggio è: 0
}
```

Although this behavior is not limited only when the `move` keyword is used
```rust
fn main() { 
	let mut count = 0; 
	let mut increment = || { 
		// Incrementiamo la variabile catturata 
		count += 1; 
		println!("Il conteggio è: {}", count); 
	}; 
	
	increment(); // Il conteggio è: 1
	// reading access for a ref mut, cannot be readed as later the cuont is modified again
	//println!("Hello, {}", count); // Il conteggio è: 0
	//^ COMPILE ERROR
	
	increment(); // Il conteggio è: 2
	
	println!("Hello, {}", count); // Il conteggio è: 2
}
```

```rust
fn main() { 
	let mut data = vec![1, 2, 3, 4, 5]; 
	
	// Definiamo una closure che può essere chiamata solo una volta 
	let mut process_data = move || { 
		data.push(6); 
		println!("Elaborazione dei dati in corso..."); 
		let sum: i32 = data.iter().sum(); 
		println!("La somma dei dati è: {}", sum); 
	}; 
	
	// Chiamiamo la closure 
	process_data(); 
	data.push(7); 
	// ^COMPILE ERROR data moved, the closure have the ownership of the data variable and thus cannot be called
}
```
## Trait that implements closures
![[Pasted image 20240429134908.png]]

Why implement the `FnOnce()`? When destroying the value, and thus cannot be called again.

```rust
fn main(){
	let mut data = vec![1,2,3];
	let mut show_and_destroy = move || {
		println!("Values: {}", data);
		drop(data);
	}
	
	show_and_destroy();
	//show_and_destroy(); 
	// cannot be called again as the data was destroyed!
}
```

## Higher order Function
```rust
fn higher_order_function<F, T, U>(f: F) where F: fn(T) -> UN {
	// code used by f(...)
}
```

```rust
fn consume_closure<F>(f:F) where F: FnOnce() -> String {
	println!("Closure of: {}", f());
}

fn main(){
	let text = "Hello World!".to_string();
	let text2 = "Hello World 2!".to_string();
		
	let printer = || text;
	let printer2 = || text2;
	
	consume_closure(printer);
	// as the function implement the trait fnOnce,
	// although the function does not drop the value 
	// cannot be called again with the same closure
	//consume_closure(printer);
	
	
	// but of course the function is not consumed
	// so i can call it for another closure
	consume_closure(printer2);
	
}
```

It is possible to return a closure from a function
```rust
fn generator(prefix: &str) -> impl FnMut() -> String { 
	let mut i = 0; 
	let b = prefix.to_string(); //"id_"
	return move || {i+=1; format!("{}{}", b, i)} 
} 

fn main() { 
	// f is a closure created by the generator with the prefix "id_"
	let mut f = generator("id_"); 
	
	for _ in 1..5 { 
		// here the closure f is called 5 times
		println!("{}",f()); // id_1 id_2 ... id_5
	} 
} 
```

# Exception
In other languages the compiler cannot identify where exception may occurs, so it remains in the programmer hands. This mean the implementation of `try{} catch{}` (in some languages also `finally{}`)
In rust it is codified with the construct `Result<T,E>` that is an `enum` with two different values:
- T: result if the operation was successful
- E: error type when something failed 

```rust
enum Result<T,E> {
	Ok(T),
	Err(E),
}
```

```rust
fn read_file(name: &str) -> Result<String,io::Error> { 
	let r1 = File::open(name); 
	let mut file = match r1 { 
		Err(why) => return Err(why), // return the error why: error generated by the opening of the file (like file does not exist or others..)
		Ok(file) => file, // actual file if no errors occur
	}; 
	
	let mut s = String::new(); 
	let r2 = file.read_to_string(&mut s); 
	
	match (r2) { 
		Err(why) => Err(why), // if reading failed
		Ok(_) => Ok(s), // result of the file into the converted into string
	} 
}
```

There are multiple methods to access and analyze the value inside of Result
- `is_ok(&self)` and `is_err(&self)` determine if the result was successful or not
- `ok(self)` and `err(self)` will transform the result into an object of type `Option<T>` or `Option<E>` from the `Result<T,E>`
	- `ok(self)`: return `Option<T>` with the successful result value (it is moved from Result, thus making it inaccessibile from it later)
	- `err(self)`: return `Option<E>` with the error value (it is moved from Result, thus making it inaccessibile from it later)
- `unwrap(self) -> T`: will return the result value, this method will invoke the macro `panic!(...)` if the result contains an error
- `unwrap_err(self) -> E`: return the error


Possible implementation:
```rust
fn divide(x: f64, y: f64) -> Result<f64, &'static str> {
	if (y == 0.0){
		Err("Cannot divide by 0")
	} else {
		Ok(x / y)
	}
}

fn main() { 
	let dividend = 10.0; 
	let divisor_success = 2.0; 
	let divisor_error = 0.0; 
	
	let result_success = divide(dividend, divisor_success).unwrap(); 
	println!("{}", result_success); 
	// if the unwrap() was used the program will panic as an error was returned
	let result_error = divide(dividend, divisor_error).unwrap_err(); 
	println!("{}", result_error); 
}
```

## Panic
the macro `panic!(...)` allows optional arguments to specify information about the error
```rust
panic!("Invalid format");
```

## Expect
the method `expect(...)` will invoke a panic with an optional argument as the panic, make the code more compact and integrate nicely with the `Result` type 

```rust
let data = result.expect("ATTENTION, ERORR WITH THE DATA");
```

## Propagate Errors
The operator `?` allow to be applied for propagating the same type of errors
It's a syntactic sugar that will allow the programmer to highlight the main execution path.
If the `?` is applied to a method that return a `Result<T, E>` inside of a function that return `Result<U, E>` where
- `E` are the same error types
- `U` and `T` **can** be the same type or different one

The `Ok` result of the function is irrelevant for the `?` keyword, it is applied only if an error arise.

```rust
fn read_file_contents(filename: &str) -> Result<String, io::Error> { 
	// Apre il file e gestisce gli errori con l'operatore ? 
	let mut file = File::open(filename)? ; 
	let mut contents = String::new(); 
	// Legge il contenuto del file e gestisce gli errori con l'operatore ? 
	file.read_to_string(&mut contents)? ; 
	// Restituisce il contenuto del file se tutto è andato bene 
	Ok(contents) 
}
```
It's important that if more error may be returned that all of them are of the same type of the Error returned by the function, otherwise the `?` will not work.
This operator does nothing more than tell the compiler to generate a match clause to capture this type of error if appears. (syntax sugar)

## Custom structure for Errors
It is possible to realize a custom error structure for the possible errors that may occur during the program execution, as an example if parsing a file with integer value:
```rust
#[derive(Debug)]
enum SumFileError{
	Io(io::Error),
	Parse(ParseIntError)
}
```

This technique require some effort as it require the implementation of the `Display`, `From` and `err` traits.

For this, is it possible to add the create `thiserr`
and let the library implement all this traits automatically (as the implementation is very much mechanic and with little modification)

```rust
#[derive(Error, Debug)] 
enum SumFileError { 
	#[error("IO error {0})"] 
	Io(#[from] io::Error), 
	#[error("Parse error {0})"] 
	Parse(#[from] ParseIntError), 
}
```

Another create is `anyhow` and is a superset of `thiserr`

This implement a more compact way to implement compact errors.
return an `anyhow::Result<T>` and use methods `with_context(|| -> )` and `context()` without closures. This will require the downcast to discriminate the specific error type returned by the `anyhow` type.

# Data Collection
Every language offers in their respective std library a set of data structure to simply the programmer works. 
As an example:
- Ordered list
- set of unique elements
- Key-value map
If there are different implementation strategies they usually were present with different performance characteristics.
Using the map as an example there are multiple way to implement them:
- `Hashmap<K,V>`
- `BTreeMap<K,V>`
- `HashSet<K,V>`
- `BTreeSet<K,V>`

Based on the type of operation the time complexity change

| Description            | Access | Research | Insert | Deletion |
| ---------------------- | ------ | -------- | ------ | -------- |
| Dynamic Array          | O(n)   | O(n)     | O(n)   | O(n)     |
| Tail with Double entry | O(n)   | O(n)     | O(1)   | O(1)     |
And so on with others implementation that have their own trade off
(The table show only the time speed, not analyzing the memory usage)
As a recap, the O notation is used to understand how much time some operation may need based on how much data need to be processed.
![[Pasted image 20240507114615.png]]

## Rust - Collection
Almost all the collection offered by the std library in #Rust have some common methods:
- `new()`
- `len()`
- `clear()`
- `is_empty()`
- `iter()`
- `extend()`

And implements the traits `IntoIterator` and `FromIterator` thus giving the methods:
- `into_iter()`
- `collect()`

### `Vec<T>`
| Description            | Access | Research | Insert | Deletion |
| ---------------------- | ------ | -------- | ------ | -------- |
| Dynamic Array          | O(n)   | O(n)     | O(n)   | O(n)     |

It can only operate on the end of the buffer with unitary time complexity operation.
`push()` to insert at the end of the buffer. If the buffer was full the all collection is moved into another free memory location with enough contiguous space left.

In general the insert has a constant time complexity (O(1)) but without consider the possible reallocation of the entire data structure. This reallocation may become really expensive but every time it doubles its size making it longer to perform but rarely to be executed after each moving operation.

To retrive the value `get(index)` and `get_mut(index)`

In addition of the standard method, the `Vec<T>` implement other methods like:
- `Vec::with_capacity(n)` allocate a `Vec` with starting capacity n
- `capacity()` return the current capacity of the `Vec`

And many others.

### `VecDeque<T>` \[Pronounced: VecDeck\] [reference](https://users.rust-lang.org/t/pronounciation-of-std-vecdeque/40617/2)
| Description            | Access | Research | Insert | Deletion |
| ---------------------- | ------ | -------- | ------ | -------- |
| Tail with Double entry | O(n)   | O(n)     | O(1)   | O(1)     |
The main difference from the `Vec` is that the `VecDeque` allows the insert and remove of an element both from the start and the end with O(1) time complexity:
- `push_back()`
- `push_front()`
- `pop_back()`
- `pop_front()`

Its implemented like a circular buffer that does not guarantee contiguous memory, this is because if the first item is removed the head would point to a free space, in this way there are no possible out of bound pointer, anyway it is possible to order with contiguous memory using the method `make_contiguous()`

![[Pasted image 20240507120438.png]]
The buffer increase in the middle reallocating when a new allocation is requested and there are no space left from the tail to the start. In this image there are 3 free slot in memory ready to be filled with data.

### `LinkedList<T>`
Allow the representation of a double linked list. This allow both moving forward and backward from any point of the list

| Description   | Access | Research | Insert | Deletion          |
| ------------- | ------ | -------- | ------ | ----------------- |
| Linked List   | O(n)   | O(n)     | O(1)   | O(1)              |
Has the same time complexity of the `VecDeque<T>`, as the insert of a new element require just the modification of 2 pointers. Like the `VecDeque` the memory is not contiguous, but unlike the `VecDeque` there are no memory allocation every time the buffer is full as the buffer does not exist, every data has a pointer to the next one and thus every location self contain the size of its own attributes and cannot exceed its own boundary.

- `split_off(index)`: after calling this method the list are split at the `index-1` and the result is two separate list
- `append()`: insert a list at the end of another list

Ordering a list is impossible, for this reason if ordered value is a requirement of the program, its usually converted into a `Vec` to allow the sorting to happen:
```rust
fn main(){
	// Convertire la LinkedList in un Vec 
	let mut vec: Vec<_> = list.into_iter().collect(); 
	
	// Ordinare il Vec 
	vec.sort(); 
	
	// Convertire il Vec ordinato in una LinkedList 
	let sorted_list: LinkedList<_> = vec.into_iter().collect();
}
```

### Map: `HashMap<K,V>`, `BTreeMap<K,V>`
- `HashMap<K,V>` store its value as a hast table to quick access single file in the heap.
	- The key `K` must implement the traits `Eq` and `Hash`
	- Very quick to find a single value, slow to find a range of value
- `BTreeMap<K,V>`: store the value as a tree representation where every value is a node
	- The key `K` must implement the trait `Ord`
	- Every node have a buffer with different value and a reference to children node
	- Slow to find a single value, very quick to find a range of value

Both use the same logic of the `Vec` and `VecDeque` for memory allocation: buffer with small size that is double and reallocated in contiguous memory every time its being filled

### `Entry<'a,K,V>`
Rust offers an optimizer for Map usage. The method `entry` let the research of a key inside of a map that return an `enum` with the result of the search. 
```rust
entry(&mut self, key: k) -> Entry<'a, K, V>

pub enum Entry<'a, K, V> {
	Occupied(OccupiedEntry<'a, K, V>)
	Vacant(VacantEntry<'a, K, V>)
}
```
The result is the position of the key if present or the position of a vacant spot where to insert that value
this allow concatenation with the method:
- `and_modify(fn -> V)`
- `or_insert(V)`

### `BinaryHeap<T>`
Collection of type `T` stored in the heap where the highest value is always stored in the front. `T` must implement the trait `Ord` 
- `peek()` return the highest value with O(1)
- `peek_mut()` same as `peek` but let you modify the value, this this with complexity O(log(n))

# Multithreading
Creating a program multithreading will require the operating system to allow the program to create thread for the operation to happens.

- Native thread: creation of the thread is require the operating system to schedule and activate the thread
- Green thread: creation of thread at user level

## Native Thread
Different operating system offers similar functionality. using standard library multiplatform it will interface to the operating system to allow thread operation.

Supported function:
+ **Creation**: need the operation to be executed with the required stack size
	+ return an handle
+ **Identification** of the current thread by a unique code at system level **TID**
+ **Waiting** the thread to be terminated by the handle and access to its final state. (join operation)

From the supported operation one important seems missing: **deletion**.
This can be implemented only inside the thread.

## Concurrency implication
Adding thread will allow temporal overlap at cost of overhead. This will help the program execution if:
- the program is **CPU bound**
- a lot of waiting: **I/O bound**
- A lot of independent instruction to be executed (e.g. division of two independent number that need lots of work to find both of them, then two thread can be used to find both in parallel and the in the original thread do the final division)

## Synchronization
If two thread try to access the same variable it will create a situation called **data race** with possible problem like non deterministic 

# Processes
#2024-06-04
A process is identified as the unit base for the execution of an application in an OS context. Precisely it is identified by the OS with a #PID (Process ID). It define an address space where one or more thread can work independent from each others.

## API
### Windows
The function `CreateProcess(...)` create a new address space with an executable image and activate a primary thread inside of him

The function is called by a process, thus creating two process, one dependent from the other. 

To end a process `ExitProcess(...)`

### Linux
The function `fork()` create a new address space (only after a write) with an identical copy of the 'father' process. Their difference is the return value. 
After the `fork()` all pages are marked with the flag **CopyOnWrite**, that means that all read will leave the same pages for both processes but when one of the two process try to write, that specific page is duplicated separating their address space.

To end a process `exit(...)`

## Process in Rust
In Rust the standard library give the programmer the module `std::process` and the struct `Command` let the creation of a new process.
The struct uses the pattern builder to configure, create and communicate with the child process.
The method `output()` will start the command
```rust
fn main(){
	Command::new("cmd")
			.args(["/C", "echo hello"])
			.output()
			.expect("failed to execute process")
}
```
