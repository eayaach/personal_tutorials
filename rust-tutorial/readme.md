# INTRODUCCIÓN PERSONAL A RUST

## comandos utiles de rust

cargo run: compila y ejecuta
cargo build: solo compila
cargo check: ve errores de compilacion
cargo test: ejecuta los tests
cargo add serde: agregar dependencias
cargo doc --open: abre la documentacion


## Flujo báscio de trabajo con rust: 
cargo new  →  editar src/  →  cargo check  →  cargo run  →  cargo build --release

# Catalogo de estudio
- Variables — entiende la inmutabilidad por defecto y el shadowing
- Ownership — el concepto más importante y único de Rust; domínalo antes de seguir
- Tipos — primitivos, tuplas, arrays y vectores
- Control de flujo — especialmente match, que es mucho más potente que un switch
- Funciones — closures e iteradores son fundamentales en código idiomático
- Structs & Enums — la base de la orientación a datos en Rust
- Manejo de errores — Result y Option en lugar de excepciones y nulls

## VARIABLES: 

En Rust, let crea variables inmutables. Esto no es una restricción sino una característica de seguridad: el compilador te protege de modificar datos que no deberías.

Para permitir mutacion hay que usar mut

```rust
    fn main() {
        let mut contador = 0;
        contador += 1;
        contador += 1;
        println!("{contador}"); // 2
    }
```

### constantes:
```rust
    const MAX_PUNTOS: u32 = 100_000; // _ como separador visual
    const PI: f64 = 3.14159;
```

### shdowing y uso de variables por scope: 
```rust
fn main() {
    let x = 5;
    let x = x + 1;       // x = 6
    let x = x * 2;       // x = 12

    {
        let x = x + 100; // x = 112 (solo dentro de este bloque)
        println!("{x}"); // 112
    }

    println!("{x}");     // 12 (el bloque interior no afectó al exterior)
}
```


### scope y drop: 
En rust cuando una variable sale de su scope su memoria se libera automaticamente eso es la base del sistema de ownership que viene despues:
```rust
fn main() {
    let a = 10; // a existe aquí

    {
        let b = 20; // b existe aquí
        println!("{a} {b}"); // OK
    } // b se libera aquí

    // println!("{b}"); // ERROR: b ya no existe
    println!("{a}"); // OK, a sigue viva
}
```

¿Necesitas mutar x varias veces según lógica compleja?
    → Caso 1: let mut x = valor_inicial; + if { x = ... }

¿Es una asignación simple basada en una condición?
    → Caso 2: let x = if condicion { a } else { b };

Interesante comportamiento en rust los ifs retornan valores por defecto.

## OWNERSHIP: 

Este es el concepto que hace especial a rust, no tiene garbage collector ni manejo manual de memoria en cambio el compilador asegura seguridad de memoria en tiempo de compilacion mediante 3 reglas: 

1. Cada valor tiene exactamente UN dueño (owner)
2. Solo puede haber un dueño a la vez
3. Cuando el dueño sale del scope, el valor se libera

Alocamiento de variables: 
```rust
let x = 5;              // Stack: tamaño fijo, copia barata
let s = String::from("hola"); // Heap: tamaño dinámico, copia costosa
```

Move: transferir ownership de una variable: 
```rust
fn main() {
    let s1 = String::from("hola");
    let s2 = s1;  // s1 se MUEVE a s2

    println!("{s1}"); // ERROR: s1 ya no es válido
    println!("{s2}"); // OK
}
```

ANTES del move:
s1 → [ ptr | len | cap ] → heap: "hola"

DESPUÉS de let s2 = s1:
s1 → INVÁLIDO (el compilador lo borra conceptualmente)
s2 → [ ptr | len | cap ] → heap: "hola"  (misma dirección)

Rust directamente invalida s1 como owner de s2 evitando el double free

también tenemos el clone:
```rust
fn main() {
    let s1 = String::from("hola");
    let s2 = s1.clone();  // copia profunda y costosa

    println!("{s1}"); // OK
    println!("{s2}"); // OK
}
```
s1 → [ ptr | len | cap ] → heap: "hola"  (dirección 0x100)
s2 → [ ptr | len | cap ] → heap: "hola"  (dirección 0x200) ← nueva

Ownership en funciones: 
```rust
fn main() {
    let s = String::from("hola");
    imprimir(s);          // s se MUEVE a la función

    println!("{s}");      // ERROR: s ya no existe aquí
}

fn imprimir(texto: String) {
    println!("{texto}");
} // texto se libera aquí

// y retornar un valor transfiere el ownership al llamador
fn main() {
    let s = crear_saludo(); // ownership llega aquí
    println!("{s}");        // OK
}

fn crear_saludo() -> String {
    String::from("hola")   // ownership sale de la función
}

// reference y borrowing = usar sin tomar ownership
fn main() {
    let s = String::from("hola");
    let largo = calcular_largo(&s); // prestamos s

    println!("{s} tiene {largo} caracteres"); // s sigue vivo
}

fn calcular_largo(s: &String) -> usize {
    s.len()
} // s NO se libera aquí, solo se devuelve el préstamo

```

Un detalle importante en rust es que por defecto las referencias son inmutables. Para modificar el valor prestado usas &mut

```rust 
fn main() {
    let mut s = String::from("hola");
    agregar_mundo(&mut s);
    println!("{s}"); // "hola mundo"
}

fn agregar_mundo(s: &mut String) {
    s.push_str(" mundo");
}
```

Resumen de ownership: 
MOVE         →  transfiere ownership, el original muere
CLONE        →  copia completa del heap, el original vive
&T           →  referencia inmutable, solo lectura
&mut T       →  referencia mutable, puede modificar

IMPORTANTE RETURN IMPLICITO:
```rust
fn crear_saludo() -> String {
    String::from("hola")   // sin ; → retorna esto
}

// Equivalente exacto con return explícito:
fn crear_saludo() -> String {
    return String::from("hola");
}
```
Es importante saber que si una variable tiene referencias "activas" es decir que más adelante en el scope siguen usandose no puedo usarla ni modificarla, su manipulación queda exclusivamente ligada a la referencia. NLL (Non-Lexical Lifetimes) 

Regla: 

Una referencia vive desde donde se crea
hasta su última línea donde aparece.
Todo lo que esté entre esos dos puntos
está bloqueado según el tipo de referencia.

## TIPOS: 

Es de tipado estatico pero tiene inferencia directa: 

- Enteros: i8, u8 = signed integer de 8 bits y unsigned de 8 bits

IMPORTANTISIMO usar siempre usize para indexar arreglos y vectores porque depende de la arquitectura en la que estamos trabajando!!!!!!

- Floats: 
```rust 
let x = 3.14 // infiere que es float 64 bits por defecto
// para especificar
let y: f32 = 3.14 // es menos preciso, ocupa menos memoria
```

- Bool and Char:

```rust 
let activo: bool = true;
let inactivo = false;

// char es Unicode completo — 4 bytes
let letra: char = 'a';
let emoji: char = '🦀';   // válido en Rust
let enie: char = 'ñ';     // también válido
```

- tuplas -> en rust permiten agrupar distintos tipos directamente
```rust 
let persona: (String, u32, bool) = (String::from("Ana"), 30, true);

// acceso por índice
println!("{}", persona.0);  // "Ana"
println!("{}", persona.1);  // 30

// destructuring — la forma más idiomática
let (nombre, edad, activo) = persona;
println!("{nombre} tiene {edad} años");

// tupla vacía — el "nada" de Rust, equivale a void
let nada: () = ();

```
- arrays -> tienen tamano fijo en el stack
```rust 
let nums: [i32; 5] = [1, 2, 3, 4, 5];  // tipo y tamaño fijos
let zeros = [0; 10];                     // [0, 0, 0, ... x10]

// acceso por índice
println!("{}", nums[0]);   // 1
println!("{}", nums[4]);   // 5

// Rust verifica índices en tiempo de ejecución
// nums[10]; → panic, no undefined behavior como en C
```

- vec -> array dinamico en el heap
```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
v.push(3);

// macro vec! para inicializar con valores
let v2 = vec![1, 2, 3, 4, 5];

// acceso
println!("{}", v2[0]);          // 1 — puede hacer panic si índice inválido

// forma segura — retorna Option<&T>
match v2.get(10) {
    Some(valor) => println!("{valor}"),
    None        => println!("índice fuera de rango"),
}

// iterar
for n in &v2 {
    println!("{n}");
}
``
- string vs &str en rust: 
```rust 
let s1: &str = "hola";           // string literal, vive en el binario
                                  // inmutable, tamaño fijo

let s2: String = String::from("hola");  // en el heap
                                         // mutable, tamaño dinámico

// convertir entre ellos
let s3: &str = &s2;              // String → &str (barato)
let s4: String = s1.to_string(); // &str → String (copia al heap)

// inferencia de tipos;
let x = 42;                  // i32
let y = 3.14;                // f64
let activo = true;           // bool
let v = vec![1, 2, 3];      // Vec<i32>

// a veces necesita ayuda
let v: Vec<i32> = Vec::new();       // sin el tipo no sabe qué Vec
let n = "42".parse::<i32>().unwrap(); // turbofish ::<> para aclarar
```

## CONTROL DE FLUJO:

- if: 
```rust 
let temperatura = 30;

let clima = if temperatura > 25 {
    "caluroso"
} else if temperatura > 15 {
    "templado"
} else {
    "frío"
};

println!("{clima}"); // "caluroso"
```
Cada rama tiene que retornar el mismo tipo sino el compilador falla 

- loop -> bucle infinito:

``` rust
let mut contador = 0;

loop {
    contador += 1;
    if contador == 5 {
        break; // sale del loop
    }
}

// loop también retorna valor con break
let resultado = loop {
    contador += 1;
    if contador == 10 {
        break contador * 2; // retorna este valor
    }
};

println!("{resultado}"); // 20

```
- while:

```rust
let mut n = 0;

while n < 5 {
    println!("{n}");
    n += 1;
}
```

- for loop la forma idiomatica de iterar
```rust 
// rango
for i in 0..5 {        // 0, 1, 2, 3, 4  (excluye el 5)
    println!("{i}");
}

for i in 0..=5 {       // 0, 1, 2, 3, 4, 5 (incluye el 5)
    println!("{i}");
}

// iterar un vector
let frutas = vec!["manzana", "pera", "uva"];

for fruta in &frutas {
    println!("{fruta}");
}

// con índice
for (i, fruta) in frutas.iter().enumerate() {
    println!("{i}: {fruta}");
}
// 0: manzana
// 1: pera
// 2: uva
```

- match: match es como un switch pero mucho mas poderos. Compara un valor contra varios patrones y ejecuta el primero que coincida
```rust 
let numero = 3;

match numero {
    1         => println!("uno"),
    2 | 3     => println!("dos o tres"),   // múltiples valores
    4..=10    => println!("entre 4 y 10"), // rango inclusivo
    _         => println!("otro"),          // wildcard, como default
}
// tambien permite retornar valores:
let puntos = 85;

let nota = match puntos {
    90..=100 => "sobresaliente",
    70..=89  => "aprobado",
    50..=69  => "suficiente",
    _        => "reprobado",
};

println!("{nota}"); // "aprobado"
//match con destructuring 
let punto = (0, -2);

match punto {
    (0, 0)  => println!("origen"),
    (x, 0)  => println!("sobre eje X en {x}"),
    (0, y)  => println!("sobre eje Y en {y}"),
    (x, y)  => println!("en ({x}, {y})"),
}
// "sobre eje Y en -2"
// if let es un match simplificado para un solo caso:
let valor: Option<i32> = Some(42);

// con match — verbose
match valor {
    Some(n) => println!("tiene {n}"),
    None    => (),  // no me importa este caso
}

// con if let — más limpio
if let Some(n) = valor {
    println!("tiene {n}");
}
```

- while let: iterador mientras exista un patron 
```rust 
let mut pila = vec![1, 2, 3];

while let Some(tope) = pila.pop() {
    println!("{tope}"); // 3, 2, 1
}
// cuando pop() retorna None, el while termina
```

## FUNCIONES:

```rust 
 /// sintaxis basica
// sin retorno
fn saludar(nombre: &str) {
    println!("Hola {nombre}");
}

// con retorno — última expresión sin ;
fn sumar(a: i32, b: i32) -> i32 {
    a + b
}

// múltiples valores de retorno con tupla
fn min_max(v: &[i32]) -> (i32, i32) {
    let mut min = v[0];
    let mut max = v[0];
    for &n in v {
        if n < min { min = n; }
        if n > max { max = n; }
    }
    (min, max)
}

let (minimo, maximo) = min_max(&[3, 1, 7, 2, 9]);
println!("{minimo} {maximo}"); // 1 9
```

- Parametros &str vs string en funciones
```rust 
// MAL — toma ownership, el caller pierde su String
fn imprimir(s: String) {
    println!("{s}");
}

// BIEN — solo pide prestado, el caller conserva su valor
fn imprimir(s: &str) {
    println!("{s}");
}

// &str acepta tanto &String como &str — es más flexible
let s1 = String::from("hola");
let s2 = "hola";

imprimir(&s1); // OK
imprimir(s2);  // OK
```
-closures: funciones anonimas son funciones sin nombre que puedes guardar en variabels y pasar como argumento. La sintaxis usa || en lugar de ()

```rust 
// función normal
fn doble(x: i32) -> i32 {
    x * 2
}

// closure equivalente
let doble = |x: i32| -> i32 { x * 2 };

// closure con inferencia de tipos — más común
let doble = |x| x * 2;

println!("{}", doble(5)); // 10

let base = 10;

// la función normal NO puede ver `base`
// el closure SÍ puede capturarla
let sumar_base = |x| x + base;

println!("{}", sumar_base(5));  // 15
println!("{}", sumar_base(20)); // 30
```

- iteradores: es rust nunca veras un for con indice manual en cambio usamos iteradores encadenados:

```rust
let numeros = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
//filter
let pares: Vec<i32> = numeros.iter()
    .filter(|&&x| x % 2 == 0)
    .cloned()
    .collect();
// [2, 4, 6, 8, 10]
//map
let dobles: Vec<i32> = numeros.iter()
    .map(|&x| x * 2)
    .collect();
// [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
// mezcla
let resultado: Vec<i32> = numeros.iter()
    .filter(|&&x| x % 2 == 0) // solo pares
    .map(|&x| x * 10)          // multiplicar por 10
    .collect();
// [20, 40, 60, 80, 100]

// fold: este reduce toda la coleccion a un valor
let suma = numeros.iter()
    .fold(0, |acumulado, &x| acumulado + x);
// 55

// atajos comunes
let suma: i32  = numeros.iter().sum();         // 55
let total      = numeros.iter().count();        // 10
let maximo     = numeros.iter().max().unwrap(); // 10
let minimo     = numeros.iter().min().unwrap(); // 1
```

- funciones que recibe closures: puedes escribir funciones que acepten un closure como parametro usnado fn

``` rust 
fn aplicar(valor: i32, operacion: impl Fn(i32) -> i32) -> i32 {
    operacion(valor)
}

let resultado = aplicar(5, |x| x * x);
println!("{resultado}"); // 25

let resultado = aplicar(10, |x| x + 100);
println!("{resultado}"); // 110
```

IMPORTANTE: (v: &[i32]) esto le permite recibir cualquier secuencia de elementos array, vector, etc. recordar esto tambien 
```rust 
// sin & la función tomaría ownership y no podrías usar el array después
min_max([3, 1, 7, 2, 9]);  // ERROR además — array no es slice
min_max(&[3, 1, 7, 2, 9]); // OK — le pasas una referencia
```

De nuevo closures para entenderlo mas en detalle:
``` rust 
// esto:
fn doble(x: i32) -> i32 {
    x * 2
}

// es exactamente lo mismo que esto:
let doble = |x: i32| -> i32 {
    x * 2
};

// y como Rust infiere tipos, puedes escribirlo así:
let doble = |x| x * 2;
```
Lo mas importante es que permiten usar variables del entorno en su definicion

```rust 
let multiplicador = 3;  // variable exterior

let multiplicar = |x| x * multiplicador; // captura multiplicador

println!("{}", multiplicar(5));  // 15
println!("{}", multiplicar(10)); // 30
```

cloned() y collect():
```rust 
let numeros = vec![1, 2, 3, 4, 5];

numeros.iter()  // produce &i32, &i32, &i32 ...

numeros.iter()
    .cloned()   // ahora produce i32, i32, i32 ... (sin &)

let pares: Vec<i32> = numeros.iter()
    .cloned()
    .filter(|x| x % 2 == 0)
    .collect(); // recién aquí se ejecuta todo y se crea el Vec
// el tipo vex<i32> le dice a collect que tipo construir
```

- fold en detalle:
```rust 
.fold(valor_inicial, |acumulado, elemento| que_hacer)
// ejemplo
let numeros = vec![1, 2, 3, 4, 5];

let suma = numeros.iter().fold(0, |acumulado, &x| acumulado + x);
//                             ^   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//                             |   esta función se ejecuta por cada elemento
//                             |
//                             empieza en 0
```
Un unwrap especial cuando no estas seguro si el output puede ser none en el iter:
```rust 
let maximo = numeros.iter().max().unwrap_or(&0); // si None, retorna 0
```

## STRUCTS Y ENUMS

- Struct recordar que permite agrupar datos relacionados. Un struct es como una ficha con campos nombrados es la forma principal de crear tipos propios en rust

```rust
struct Persona {
    nombre: String,
    edad: u32,
    activo: bool,
}

// crear una instancia
let p = Persona {
    nombre: String::from("Ana"),
    edad: 30,
    activo: true,
};

// acceder a los campos
println!("{}", p.nombre); // "Ana"
println!("{}", p.edad);   // 30
// para modificarlo se debe usar mut
let mut p = Persona {
    nombre: String::from("Ana"),
    edad: 30,
    activo: true,
};

p.edad = 31; // OK

```
- impl permite agregar metodos al struct 

```rust
struct Rectangulo {
    ancho: f64,
    alto: f64,
}

impl Rectangulo {
    // método constructor — por convención se llama new
    fn new(ancho: f64, alto: f64) -> Self {
        Rectangulo { ancho, alto }
    }

    // método de instancia — &self para leer
    fn area(&self) -> f64 {
        self.ancho * self.alto
    }

    // método de instancia — &mut self para modificar
    fn escalar(&mut self, factor: f64) {
        self.ancho *= factor;
        self.alto  *= factor;
    }

    // método asociado — no recibe self, es como un static
    fn cuadrado(lado: f64) -> Self {
        Rectangulo { ancho: lado, alto: lado }
    }
}

let mut r = Rectangulo::new(10.0, 5.0);
println!("{}", r.area()); // 50.0

r.escalar(2.0);
println!("{}", r.area()); // 200.0

let c = Rectangulo::cuadrado(4.0);
println!("{}", c.area()); // 16.0
```

&self      →  solo lee, no modifica
&mut self  →  puede modificar los campos
Sin self   →  método asociado, se llama con :: no con .

- Struct update syntax: es cuando quieres crear una instancia basada en otra cambaindo solo algunos campos

```rust 
let p1 = Persona {
    nombre: String::from("Ana"),
    edad: 30,
    activo: true,
};

let p2 = Persona {
    nombre: String::from("Luis"), // solo cambiamos esto
    ..p1                          // el resto viene de p1
};
```

- Enums los usamos para representar variantes

``` rust
enum Direccion {
    Norte,
    Sur,
    Este,
    Oeste,
}

let dir = Direccion::Norte;

match dir {
    Direccion::Norte => println!("vas al norte"),
    Direccion::Sur   => println!("vas al sur"),
    Direccion::Este  => println!("vas al este"),
    Direccion::Oeste => println!("vas al oeste"),
}
```

En rust los enums pueden contener datos distintos:
```rust
enum Mensaje {
    Salir,                        // sin datos
    Mover { x: i32, y: i32 },    // con campos nombrados
    Escribir(String),             // con un String
    Color(u8, u8, u8),           // con tres valores
}

let m1 = Mensaje::Salir;
let m2 = Mensaje::Mover { x: 10, y: 20 };
let m3 = Mensaje::Escribir(String::from("hola"));
let m4 = Mensaje::Color(255, 0, 128);

match m2 {
    Mensaje::Salir              => println!("saliendo"),
    Mensaje::Mover { x, y }    => println!("moviendo a {x},{y}"),
    Mensaje::Escribir(texto)   => println!("escribiendo: {texto}"),
    Mensaje::Color(r, g, b)    => println!("color: {r},{g},{b}"),
}
// "moviendo a 10,20"
```
A los enums tambien se les puede asociar funciones con impl:
```rust
impl Mensaje {
    fn ejecutar(&self) {
        match self {
            Mensaje::Salir            => println!("saliendo"),
            Mensaje::Escribir(texto)  => println!("{texto}"),
            _                         => println!("otro mensaje"),
        }
    }
}

let m = Mensaje::Escribir(String::from("hola"));
m.ejecutar(); // "hola"
```

- Option y result: estos son los enums mas importantes en rust
```rust
// Option — cuando algo puede o no existir
enum Option<T> {
    Some(T),  // hay un valor
    None,     // no hay nada
}

// Result — cuando algo puede fallar
enum Result<T, E> {
    Ok(T),    // éxito con valor
    Err(E),   // error con descripción
}

```

- traits: un trait define comportamiento que un tipo puede implementar, es similar a una interfaz en otros lenguajes

``` rust
trait Describir {
    fn describir(&self) -> String;
}

struct Perro {
    nombre: String,
}

struct Gato {
    nombre: String,
}

impl Describir for Perro {
    fn describir(&self) -> String {
        format!("{} es un perro", self.nombre)
    }
}

impl Describir for Gato {
    fn describir(&self) -> String {
        format!("{} es un gato", self.nombre)
    }
}
```
- Polimorfismo en rust: existe en dos formas 

Forma 1: estatica con impl trait resuelta en compilacion mas rapida.
```rust
// acepta cualquier tipo que implemente Describir
fn imprimir(animal: &impl Describir) {
    println!("{}", animal.describir());
}

let perro = Perro { nombre: String::from("Rex") };
let gato  = Gato  { nombre: String::from("Michi") };

imprimir(&perro); // "Rex es un perro"
imprimir(&gato);  // "Michi es un gato"
```

Forma 2: dinamica con dyn trait (es resuelta en ejecucion)
```rust 
// Vec con animales de distintos tipos
let animales: Vec<Box<dyn Describir>> = vec![
    Box::new(Perro { nombre: String::from("Rex") }),
    Box::new(Gato  { nombre: String::from("Michi") }),
];

for animal in &animales {
    println!("{}", animal.describir());
}
// "Rex es un perro"
// "Michi es un gato"
```

que es box?
Box es simplemente un puntero al heap. Lo necesitas aquí porque Perro y Gato pueden tener tamaños distintos en memoria, y un Vec necesita saber el tamaño de cada elemento. Box<dyn Animal> siempre tiene el mismo tamaño — es solo un puntero:

Sin Box:
Vec<dyn Animal> → ERROR — ¿cuánto espacio reservo por elemento?
                           Perro y Gato pueden pesar distinto

Con Box:
Vec<Box<dyn Animal>> → OK — cada elemento es un puntero (tamaño fijo)
                             el puntero apunta al Perro o Gato en el heap


```rust
// comparacion final entre los tipos de polimorfismo

// ESTÁTICO — impl Trait
// cada tipo tiene su propia versión de la función
// más rápido, pero no puedes mezclar tipos
fn hacer_sonar_estatico(animal: &impl Animal) {
    println!("{}", animal.sonido());
}

hacer_sonar_estatico(&Perro); // OK
hacer_sonar_estatico(&Gato);  // OK
// pero no puedes hacer un Vec con ambos


// DINÁMICO — dyn Trait
// una sola versión de la función para todos
// un poco más lento, pero puedes mezclar tipos
fn hacer_sonar_dinamico(animal: &dyn Animal) {
    println!("{}", animal.sonido());
}

let animales: Vec<Box<dyn Animal>> = vec![
    Box::new(Perro),
    Box::new(Gato),
];
for a in &animales {
    hacer_sonar_dinamico(a.as_ref()); // mismo código, tipos distintos
}
```

Cuándo usar cada uno
impl Trait  →  cuando la función recibe UN solo tipo a la vez
               y no necesitas mezclar tipos en colecciones
               es la opción por defecto — más rápida

dyn Trait   →  cuando necesitas mezclar tipos distintos
               en un Vec, HashMap, o cualquier colección
               o cuando el tipo se decide en tiempo de ejecución

- overriding existe pero con metodos por defecto aca un trati puede tener una implementacion por defecto que los tipos pueden sobreescribir o no:
```rust
trait Saludar {
    fn nombre(&self) -> &str;

    // implementación por defecto
    fn saludar(&self) {
        println!("Hola, soy {}", self.nombre());
    }
}

struct Persona {
    nombre: String,
}

impl Saludar for Persona {
    fn nombre(&self) -> &str {
        &self.nombre
    }
    // no sobreescribimos saludar — usa el por defecto
}

struct Robot {
    id: u32,
}

impl Saludar for Robot {
    fn nombre(&self) -> &str {
        "Robot"
    }

    // sobreescribimos saludar
    fn saludar(&self) {
        println!("BEEP. SOY ROBOT #{}", self.id);
    }
}

let p = Persona { nombre: String::from("Ana") };
let r = Robot { id: 42 };

p.saludar(); // "Hola, soy Ana"   — usa el default
r.saludar(); // "BEEP. SOY ROBOT #42" — overrideado
```

Resumen de option:
```rust
let con_valor: Option<i32> = Some(42);
let sin_valor: Option<i32> = None;

// unwrap_or — valor por defecto si es None
let a = con_valor.unwrap_or(0); // 42
let b = sin_valor.unwrap_or(0); // 0

// map — transforma el valor si es Some, no toca None
let doble = con_valor.map(|x| x * 2); // Some(84)
let nada  = sin_valor.map(|x| x * 2); // None

// is_some / is_none — verificar sin extraer
if con_valor.is_some() {
    println!("tiene valor");
}

// if let — la forma más idiomática de extraer
if let Some(n) = con_valor {
    println!("el valor es {n}"); // "el valor es 42"
}

// match — cuando necesitas manejar ambos casos
match sin_valor {
    Some(n) => println!("tengo {n}"),
    None    => println!("no hay nada"),
}

// caso real buscar en un vector
let usuarios = vec!["Ana", "Luis", "María"];

let encontrado = usuarios.iter().find(|&&u| u == "Luis");

match encontrado {
    Some(nombre) => println!("encontré a {nombre}"),
    None         => println!("no existe"),
}
```
Resumen Result: Result<T, E> se usa para operaciones que pueden fallar T es el tipo de exito y E el ttipo de error
```rust 
// ejemplo básico — parsear un número
let ok:  Result<i32, _> = "42".parse::<i32>();   // Ok(42)
let err: Result<i32, _> = "hola".parse::<i32>(); // Err(...)

// unwrap_or — valor por defecto si falla
let n = "hola".parse::<i32>().unwrap_or(0); // 0

// match — manejar ambos casos
match "42".parse::<i32>() {
    Ok(n)  => println!("número: {n}"),
    Err(e) => println!("error: {e}"),
}

// if let — cuando solo te importa el Ok
if let Ok(n) = "42".parse::<i32>() {
    println!("parseado: {n}");
}

// map — transformar el Ok sin tocar el Err
let resultado = "5".parse::<i32>()
    .map(|n| n * 2); // Ok(10)
```
operador ? para propagar errores es la forma mas idiomatica de manejar errores en rust

```rust
use std::fs;
use std::io;

fn leer_archivo(path: &str) -> Result<String, io::Error> {
    let contenido = fs::read_to_string(path)?; // si falla, retorna Err
    let mayusculas = contenido.to_uppercase();  // si llegamos acá, Ok
    Ok(mayusculas)
}

match leer_archivo("datos.txt") {
    Ok(texto) => println!("{texto}"),
    Err(e)    => println!("error al leer: {e}"),
}
// sin ? tendriamos que hacerlo asi
fn leer_archivo(path: &str) -> Result<String, io::Error> {
    let contenido = match fs::read_to_string(path) {
        Ok(c)  => c,
        Err(e) => return Err(e), // el ? hace exactamente esto
    };
    Ok(contenido.to_uppercase())
}
```

## MANEJO DE ERRORES:

Rust no tiene excepciones. En cambio tiene dos mecanismos formales: `Option` y `Result`.

---

## Dos categorías de errores

```
panic!  →  errores irrecuperables, el programa muere
Result  →  errores recuperables, el programa decide qué hacer
```

---

## panic! — cuándo usarlo

`panic!` detiene el programa inmediatamente. Ocurre automáticamente en situaciones como índice fuera de rango o `unwrap()` sobre `None`:

```rust
let v = vec![1, 2, 3];
v[10]; // panic automático — índice inválido

let x: Option<i32> = None;
x.unwrap(); // panic automático

// panic manual
panic!("algo salió muy mal");
```

Úsalo solo cuando el error es **imposible de recuperar** — un bug en la lógica, un estado que nunca debería ocurrir. Para todo lo demás usa `Result`.

---

## Result — el centro del manejo de errores

```rust
enum Result<T, E> {
    Ok(T),   // éxito — contiene el valor
    Err(E),  // fallo  — contiene el error
}
```

### Métodos útiles

```rust
let ok:  Result<i32, &str> = Ok(42);
let err: Result<i32, &str> = Err("algo falló");

// extraer el valor
ok.unwrap()              // 42, panic si es Err
ok.unwrap_or(0)          // 42, o 0 si es Err
ok.unwrap_or_else(|e| {  // 42, o ejecuta el closure si es Err
    println!("error: {e}");
    0
})

// verificar
ok.is_ok()               // true
err.is_err()             // true

// transformar sin extraer
ok.map(|n| n * 2)        // Ok(84)  — transforma el Ok
err.map(|n| n * 2)       // Err("algo falló") — no toca el Err

ok.map_err(|e| format!("ERROR: {e}"))  // no toca el Ok
err.map_err(|e| format!("ERROR: {e}")) // Err("ERROR: algo falló")

// encadenar operaciones que también retornan Result
ok.and_then(|n| {
    if n > 0 { Ok(n * 2) } else { Err("negativo") }
}) // Ok(84)
```

---

## El operador `?` — la herramienta más importante

Cuando pones `?` después de un `Result`:

- Si es `Ok` — extrae el valor y continúa
- Si es `Err` — retorna el error inmediatamente de la función

```rust
use std::fs;
use std::io;

fn leer_y_procesar(path: &str) -> Result<usize, io::Error> {
    let contenido = fs::read_to_string(path)?; // si falla, retorna Err
    let lineas    = contenido.lines().count();  // si llegamos acá, Ok
    Ok(lineas)
}
```

### Encadenando múltiples operaciones

```rust
fn procesar_archivo(path: &str) -> Result<String, io::Error> {
    let contenido  = fs::read_to_string(path)?;   // falla → retorna Err
    let procesado  = contenido.trim().to_string(); // ok → continúa
    fs::write("salida.txt", &procesado)?;          // falla → retorna Err
    Ok(procesado)                                  // todo ok → retorna Ok
}
```

Sin `?` cada línea sería un `match` anidado — el código se volvería ilegible rápidamente.

---

## Crear tus propios errores

Para proyectos reales necesitas definir tus propios tipos de error con un enum:

```rust
#[derive(Debug)]
enum ErrorApp {
    ArchivoNoEncontrado(String),
    FormatoInvalido(String),
    PermisosDenegados,
}

// implementar Display para mensajes legibles
impl std::fmt::Display for ErrorApp {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ErrorApp::ArchivoNoEncontrado(path) =>
                write!(f, "archivo no encontrado: {path}"),
            ErrorApp::FormatoInvalido(detalle) =>
                write!(f, "formato inválido: {detalle}"),
            ErrorApp::PermisosDenegados =>
                write!(f, "permisos denegados"),
        }
    }
}

fn abrir_config(path: &str) -> Result<String, ErrorApp> {
    if path.is_empty() {
        return Err(ErrorApp::ArchivoNoEncontrado(path.to_string()));
    }
    Ok(String::from("contenido del archivo"))
}

match abrir_config("") {
    Ok(contenido) => println!("{contenido}"),
    Err(e)        => println!("Error: {e}"), // "Error: archivo no encontrado: "
}
```

---

## Convertir entre tipos de error con `From`

Cuando una función usa `?` con errores de distintos tipos, necesitas convertirlos a un tipo común:

```rust
#[derive(Debug)]
enum ErrorApp {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
}

impl From<std::io::Error> for ErrorApp {
    fn from(e: std::io::Error) -> Self {
        ErrorApp::Io(e)
    }
}

impl From<std::num::ParseIntError> for ErrorApp {
    fn from(e: std::num::ParseIntError) -> Self {
        ErrorApp::Parse(e)
    }
}

fn leer_numero(path: &str) -> Result<i32, ErrorApp> {
    let contenido = std::fs::read_to_string(path)?;  // io::Error → ErrorApp::Io
    let numero    = contenido.trim().parse::<i32>()?; // ParseIntError → ErrorApp::Parse
    Ok(numero)
}
```

El `?` llama automáticamente a `From` para convertir el error al tipo que la función retorna.

---

## Option y Result juntos

A veces necesitas convertir entre ellos:

```rust
// Option → Result
let x: Option<i32> = Some(42);
let r: Result<i32, &str> = x.ok_or("no había valor"); // Ok(42)

let y: Option<i32> = None;
let r: Result<i32, &str> = y.ok_or("no había valor"); // Err("no había valor")

// Result → Option
let ok:  Result<i32, &str> = Ok(42);
let opt: Option<i32> = ok.ok(); // Some(42)

let err: Result<i32, &str> = Err("fallo");
let opt: Option<i32> = err.ok(); // None
```

---

## Patrón completo — aplicación real

```rust
#[derive(Debug)]
enum Error {
    Io(std::io::Error),
    NumeroInvalido(String),
}

impl std::fmt::Display for Error {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            Error::Io(e)             => write!(f, "error de IO: {e}"),
            Error::NumeroInvalido(s) => write!(f, "número inválido: {s}"),
        }
    }
}

impl From<std::io::Error> for Error {
    fn from(e: std::io::Error) -> Self { Error::Io(e) }
}

fn sumar_archivo(path: &str) -> Result<i32, Error> {
    let contenido = std::fs::read_to_string(path)?;

    let suma = contenido
        .lines()
        .map(|linea| {
            linea.trim().parse::<i32>()
                .map_err(|_| Error::NumeroInvalido(linea.to_string()))
        })
        .collect::<Result<Vec<i32>, Error>>()?
        .iter()
        .sum();

    Ok(suma)
}

fn main() {
    match sumar_archivo("numeros.txt") {
        Ok(suma) => println!("suma total: {suma}"),
        Err(e)   => println!("falló: {e}"),
    }
}
```

---

## Resumen

| Herramienta | Cuándo usarla |
|---|---|
| `panic!` | Error irrecuperable, bug en la lógica |
| `Result<T, E>` | Operación que puede fallar |
| `Option<T>` | Valor que puede no existir |
| `?` | Propagar el error automáticamente |
| `unwrap()` | Solo en ejemplos o cuando es imposible que falle |
| `unwrap_or(val)` | Extraer o retornar valor por defecto |
| `map()` | Transformar el Ok/Some sin extraer |
| `and_then()` | Encadenar operaciones que retornan Result |
| `ok_or()` | Convertir Option en Result |
| `From` | Convertir entre tipos de error automáticamente |

