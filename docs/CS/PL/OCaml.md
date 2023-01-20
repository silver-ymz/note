# OCaml

> 注: 本文档学习时基于 OCaml 4.14.1, 标准库使用 [Base 0.15.1](https://ocaml.org/p/base/v0.15.1/doc/Base/index.html)


## 基本语法

- Ocaml强调整型与浮点型的不同,如:
```OCaml
3 + 4;;  (* - : int = 7 *)
3. +. 4.;;  (* - : float = 7 *)
```

- OCaml使用`let`来定义变量和函数, `let rec`定义递归函数, `fun`定义匿名函数, 如:
```OCaml
let x = 3;;  (* val x : int = 3 *)
let f x = x + 1;;  (* val f : int -> int = <fun> *)
let rec factorial x = if x = 0 then 1 else x * factorial (x - 1);;
(* val factorial : int -> int = <fun> *)
let g = fun x -> x + 1;;  (* val g : int -> int = <fun> *)
```

- OCaml区分Error和Exception,如:
```OCaml
let add_potato x = x + "patato";;
(* Error: This expression has type string but an expression was expected of type int *)

let is_a_multiple_of_3 x y = x % y = 0;;
is_a_multiple 8 0;;
(* Exception: Invalid_argument "8 % 0 in core_int.ml: modulus should be positive". *)
```

- 模式匹配, 分支, 循环, 异常处理:
```OCaml
let f x = match x with
  | 0 -> "zero"
  | 1 -> "one"
  | _ -> "other";;
(* val f : int -> string = <fun> *)
let f x = if x > 0 then "positive" else "negative";;
(* val f : int -> string = <fun> *)
let f x = for i = 0 to x do print_int i done;;
(* val f : int -> unit = <fun> *)
let f x = while x > 0 do print_int x; x = x - 1 done;;
(* val f : int -> unit = <fun> *)
let f x = try x / 0 with Division_by_zero -> 0;;
(* val f : int -> int = <fun> *)
```

- OCaml使用`;;`来分隔表达式, `;`来分隔语句.

- OCaml默认变量是不可变的, 使用`ref`来创建可变变量, 详见[Refs](#refs).


## 基本数据类型

### Tuples
```OCaml
let a_tuple = (3, "three");;
(* val a_tuple : int * string = (3, "three") *)
let (x,y) = a_tuple;;
(* x = 3, y = "three" *)
```

### Options

```OCaml
let none = None;;  (* val none : 'a option = None *)
let some = Some 3;;  (* val some : int option = Some 3 *)
```

基本的Option操作:
```OCaml
val is_none : 'a option -> bool
val is_some : 'a option -> bool

val some : 'a -> 'a option
val value : 'a option -> default:'a -> 'a
```

### Lists
```OCaml
let languages = ["OCaml"; "Perl"; "C"];;
(* val languages : string list = ["OCaml"; "Perl"; "C"] *)
let languages = "OCaml" :: "Perl" :: "C" :: [];;
(* val languages : string list = ["OCaml"; "Perl"; "C"] *)
```

基本的List操作:
```OCaml
val cons : 'a -> 'a list -> 'a list  (* 与 :: 相同 *)
val append : 'a list -> 'a list -> 'a list (* 与 @ 相同 *)

val hd : 'a list -> 'a option
val tl : 'a list -> 'a list option
val last : 'a list -> 'a option
val nth : 'a list -> int -> 'a option

val is_empty : 'a list -> bool
val length : 'a list -> int

val rev : 'a list -> 'a list
val sort : 'a list -> compare:( 'a -> 'a -> int ) -> 'a list

val iter : 'a list -> f:( 'a -> unit ) -> unit
val map : 'a list -> f:( 'a -> 'b ) -> 'b t

val find : 'a list -> f:( 'a -> bool ) -> 'a option
val filter : 'a list -> f:( 'a -> bool ) -> 'a list
val exists : 'a list -> f:( 'a -> bool ) -> bool
val for_all : 'a list -> f:( 'a -> bool ) -> bool

val find_map : 'a list -> f:( 'a -> 'b option ) -> 'b option
val filter_map : 'a list -> f:( 'a -> 'b option ) -> 'b t

val fold : 'a list -> init:'accum -> f:( 'accum -> 'a -> 'accum ) -> 'accum
val fold_right : 'a list -> init:'accum -> f:( 'a -> 'accum -> 'accum ) -> 'accum

val concat : 'a list t -> 'a list
val init : int -> f:( int -> 'a ) -> 'a list

val compare : ( 'a -> 'a -> int ) -> 'a list -> 'a list -> int
val min_elt : 'a t -> compare:( 'a -> 'a -> int ) -> 'a option
val max_elt : 'a t -> compare:( 'a -> 'a -> int ) -> 'a option
```

### Arrays

```OCaml
let a = [| 1; 2; 3 |];;
(* val a : int array = [|1; 2; 3|] *)
let a = Array.create ~len:3 0;;
(* val a : int array = [|0; 0; 0|] *)
let a = Array.init 3 ~f:(fun i -> i);;
(* val a : int array = [|0; 1; 2|] *)
```

基本的Array操作:
```OCaml
val create : len:int -> 'a -> 'a array
val init : int -> f:( int -> 'a ) -> 'a array

val is_empty : 'a array -> bool
val length : 'a array -> int

val get : 'a array -> int -> 'a  (* 与 .(i) 相同 *)
val set : 'a array -> int -> 'a -> unit  (* 与 .(i) <- x 相同 *)
(* 若访问越界,则抛出异常 Invalid_argument "index out of bounds" *)

val rev : 'a t -> 'a t
val append : 'a t -> 'a t -> 'a t

val iter : 'a array -> f:( 'a -> unit ) -> unit
val map : 'a array -> f:( 'a -> 'b ) -> 'b array
val find : 'a array -> f:( 'a -> bool ) -> 'a option
val filter : 'a array -> f:( 'a -> bool ) -> 'a array
val exists : 'a array -> f:( 'a -> bool ) -> bool
val for_all : 'a array -> f:( 'a -> bool ) -> bool
val find_map : 'a array -> f:( 'a -> 'b option ) -> 'b option
val filter_map : 'a array -> f:( 'a -> 'b option ) -> 'b array


val fold : 'a array -> init:'accum -> f:( 'accum -> 'a -> 'accum ) -> 'accum
val fold_right : 'a array -> init:'accum -> f:( 'a -> 'accum -> 'accum ) -> 'accum

val sort : ?pos:int -> ?len:int -> 'a t -> compare:( 'a -> 'a -> int ) -> unit
val stable_sort : 'a array -> compare:( 'a -> 'a -> int ) -> unit
val is_sorted : 'a t -> compare:( 'a -> 'a -> int ) -> bool

val to_list : 'a array -> 'a list
val of_list : 'a list -> 'a array
```

### Refs

```OCaml
let r = { contents = 0 };;
(* val r : int ref = {contents = 0} *)
let r = ref 0;;
(* val r : int ref = {Base.Ref.contents = 0} *)
let r = Ref.create 0;;
(* val r : int ref = {Base.Ref.contents = 0} *)

let x = !r;;
(* val x : int = 0 *)
r :=  !r + 3;;
(* - : unit = () *)
```

基本的Ref操作:
```OCaml
val create : 'a -> 'a ref
val (!) : 'a ref -> 'a
val (:=) : 'a ref -> 'a -> unit
val replace : 'a ref -> ( 'a -> 'a ) -> unit
```