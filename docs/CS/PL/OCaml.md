# OCaml

学习时的阅读资料:

  1. [Real World OCaml](https://dev.realworldocaml.org/)

  2. [OCaml.org](https://ocaml.org/learn/tutorials/index.html)

  3. [OCaml Reference Manual](https://v2.ocaml.org/manual/index.html)

???+ info "OCaml版本信息"
     本文档学习时基于 OCaml 4.14.1, 标准库使用 [Base 0.15.1](https://ocaml.org/p/base/v0.15.1/doc/Base/index.html)


## 基本语法

- 强调整型与浮点型的不同,如:
```OCaml
3 + 4;;  (* - : int = 7 *)
3. +. 4.;;  (* - : float = 7 *)
```

- 使用`let`来定义变量和函数, 如:
```OCaml
let x = 3;;  (* val x : int = 3 *)
let f x = x + 1;;  (* val f : int -> int = <fun> *)
```

- 区分Error和Exception,如:
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

- 使用`;;`来分隔表达式, `;`来分隔语句.

- 默认变量是不可变的, 可以使用`ref`来创建可变变量, 详见[Refs](#refs).

### 函数

#### 匿名函数

使用`fun`来定义匿名函数, 如:
```OCaml
let f = fun x -> x + 1;;
(* val f : int -> int = <fun> *)
```

#### 中缀函数

当函数名称包含以下字符时, 可以使用中缀函数`~ ! $ % & * + - . / : < = > ? @ ^ |`(第一个字不为`~ ! $`), 如:
```OCaml
let (+!) (x1,y1) (x2,y2) = (x1 + x2, y1 + y2);;
(* val ( +! ) : int * int -> int * int -> int * int = <fun> *)
(3,2) +! (-2,4);;  (* - : int * int = (1, 6) *)
```

#### 递归函数

使用`let rec`来定义递归函数, 如:
```OCaml
let rec factorial x = if x = 0 then 1 else x * factorial (x - 1);;
(* val factorial : int -> int = <fun> *)
```

#### `function`关键词

可以使用`function`关键词来定义函数, 其不支持多值函数, 但内置模式匹配, 如:
```OCaml
let f = function
  | 0 -> "zero"
  | 1 -> "one"
  | _ -> "other";;
(* val f : int -> string = <fun> *)
```

#### 多参数函数

所有多参数函数均满足柯里化, 如:
```OCaml
let f = (+) 1;;  (* val f : int -> int = <fun> *)
f 2;;  (* - : int = 3 *)
```

#### 标记参数

函数参数可以通过标记(`~`)来指定, 如:
```OCaml
let f ~x ~y = x + y;;
(* val f : x:int -> y:int -> int = <fun> *)
f ~x:3 ~y:4;;  (* - : int = 7 *)
let x = 3 in
  let y = 4 in
    f ~x ~y;;  (* - : int = 7 *)
```

以下情形, 常用标记参数:

- 函数参数列表较长时, 如:
```OCaml
val f : x:int -> y:int -> z:int -> int
```

- 函数参数类型无法提供足够的信息时, 如:
```OCaml
val create_hashtable :
  init_size:int -> allow_shrinking:bool -> ('a,'b) Hashtable.t
```

- 函数参数类型相同时, 需要消除歧义, 如:
```OCaml
val substring: string -> pos:int -> len:int -> string
```

- 需要灵活的参数顺序或常作为高阶函数使用时, 如:
```OCaml
val List.iter : 'a list -> f:( 'a -> unit ) -> unit
```

#### 可选参数

函数参数可以设置为可选(`?`), 如:
```OCaml
let concat ?sep x y =
  let sep = match sep with None -> "" | Some s -> s in
  x ^ sep ^ y;;
(* val concat : ?sep:string -> string -> string -> string = <fun> *)
concat "foo" "bar"  (* without the optional argument *);;
(* - : string = "foobar" *)
concat ~sep:":" "foo" "bar"  (* with the optional argument *);;
(* - : string = "foo:bar" *)
```

- 可选参数可以提供默认值, 如:
```OCaml
let concat ?(sep="") x y = x ^ sep ^ y;;
(* val concat : ?sep:string -> string -> string -> string = <fun> *)
```

- 可选参数可以显式传递, 如:
```OCaml
concat ~sep:":" "foo" "bar" (* provide the optional argument *);;
concat ?sep:(Some ":") "foo" "bar" (* pass an explicit [Some] *);
concat ?sep:None "foo" "bar" (* explicitly pass `None` *);;
```

#### 参数擦除

在部分应用时, 函数会优先匹配位置参数, 当匹配到可选参数后的位置参数时, 可选参数会被擦除, 如:
```OCaml
concat ~sep:":";;  (* - : string -> string -> string = <fun> *)
concat ":";;  (* - : string -> string = <fun> *)
let concat x ?(sep="") y = x ^ sep ^ y;;
concat ":";;  (* - : ?sep:string -> string -> string *)
```

> 当可选参数位于末尾时, 由于无法擦除(即必须显式传递), 编译器会报Warning

### 模块(Module)

当项目较为复杂时, 可以将每个文件视为一个模块, 通过模块来组织代码.

一个模块分为声明(.mli)与实现(.ml). 声明文件用于描述模块的接口, 实现文件用于实现模块的功能. 例如:
```OCaml title="my_module.mli"
type t
val map_find : t -> key -> value option
```

```OCaml title="my_module.ml"
type t = (key * value) list
let map_find map key =
  try Some (List.assoc key map)
  with Not_found -> None
```

#### 模块签名(Signature)

模块签名用于描述模块的接口, 如:
```OCaml
module type MapModule = sig
  type t
  val map_find : t -> key -> value option
end
```
模块签名可以与模块实现进行匹配, `module <name> : <sign> = <impl>` 如:
```OCaml
module MyModule : MapModule = struct
  type t = (key * value) list
  let map_find map key =
    try Some (List.assoc key map)
    with Not_found -> None
end
```

#### 模块展开(Open)

模块展开用于将模块中的所有内容展开到当前作用域, 如:
```OCaml
module M = struct let foo = 3 end;;
foo;;  (* Error: Unbound value foo *)
open M;;
foo;;  (* - : int = 3 *)
```

在实际开发中, 应尽量减少直接展开模块以避免命名冲突. 展开模块时, 也应选择设计为展开的模块, 如`Base`, `Base.Option.Monad_infix`, `Base.Float.O`

#### 局部展开

局部展开用于将模块暂时性展开到当前作用域, 以避免命名冲突, 如:
```OCaml
let average x y =
  let open Int64 in
  (x + y) / of_int 2;;
```

#### 模块别名

若模块名过长, 可以使用模块别名以减少代码量, 如:
```OCaml
let average x y =
  let module I64 = Int64 in
  (x + y) / I64.of_int 2;;
```

#### 模块包含(Include)

模块包含可以用于拓展已有模块的功能, 如:
```OCaml
module Interval = struct
  type t = | Interval of int * int
           | Empty

  let create low high = 
    if low > high then Empty
    else Interval (low, high)
end;;

module ExtendedInterval = struct
  include Interval

  let contains t x =
    match t with
    | Empty -> false
    | Interval (low, high) -> low <= x && x <= high
end;;
```


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
val filter : 'a list -> f:( 'a -> bool ) -> 'a list

val fold : 'a list -> init:'accum -> f:( 'accum -> 'a -> 'accum ) -> 'accum
val reduce : 'a list -> f:( 'a -> 'a -> 'a ) -> 'a option

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

基本的 Array 操作:
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

### String

```OCaml
let s = "hello";;
(* val s : string = "hello" *)
let s = String.make 5 ' ';;
(* val s : string = "     " *)
let s = String.init 5 ~f:(fun i -> Char.of_int_exn (i + 97));;
(* val s : string = "abcde" *)
```

基本的 String 操作(大多数与 List 相同):
```OCaml
val make : int -> char -> string
val escaped : string -> string
val copy : string -> string

val uppercase : string -> string
val lowercase : string -> string
val capitalize : string -> string
val uncapitalize : string -> string

val is_substring : string -> ~substring:string -> bool
val is_prefix : string -> ~prefix:string -> bool
val is_suffix : string -> ~suffix:string -> bool
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

