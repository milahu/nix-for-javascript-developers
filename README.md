## How to run examples

Use `nix eval --expr` instead of `nix-instantiate --eval --expr`:

```bash
$ nix-instantiate --eval --expr '(x: y: { a = x + "-" + y; }) "a" "b"'
{ a = <CODE>; }
$ nix eval --expr '(x: y: { a = x + "-" + y; }) "a" "b"'
{ a = "a-b"; }
```

For node you must use `console.log` unless it is already in the example.

```bash
$ node -e 'console.log(((x, y) => ({ a: x + "-" + y }))("a", "b"))'
{ a: 'a-b' }
```

If you want to have `nix eval` available in nixos stable, you need to install unstable nix command as described in https://nixos.wiki/wiki/Flakes.

To see complex values in the Nix REPL, use `lib.traceValSeqN`

```
nix-repl> lib.traceValSeqN 2 [[1 [2]]]
trace: [
  [
    1
    […]
  ]
]
[ [ ... ] ]
```

## Examples

| Nix | Javascript |
| --- | --- |
| `"Hello world!"` | `"Hello world!"` |
| `if true then 3 else 2` | `true ? 3: 2` |
| `builtins.typeOf ""` | `typeof ""` |

### Numbers

| Nix | Javascript |
| --- | --- |
| `2/3` | `require("path").join("2", "3")` |
| `2/ 3` | `Math.floor(2/3)` | |
| `2/ 3.` | `2/3` |
| `builtins.bitAnd 2 3` | `2 & 3` |
| `builtins.bitOr 2 3` | `2 \| 3` |
| `builtins.bitXor 2 3` | `2 ^ 3` |
| `lib.strings.charToInt "a"` | `"a".charCodeAt(0)` |
| `lib.strings.toInt "1"` | `parseInt("1")` |
| `let toFloat = s: if builtins.match "[[:space:]]*(-?([[:digit:]]+\\.[[:digit:]]*|[[:digit:]]*\\.[[:digit:]]+|[[:digit:]]+)([eE][[:digit:]]+)?)[[:space:]]*" s != null then builtins.fromJSON s else throw "not a number: ${builtins.toJSON s}"; in toFloat "-1.234e50"` | `parseFloat("-1.234e50")` |
| `lib.min 1 2` | `Math.min(1, 2)` |
| `lib.max 1 2` | `Math.max(1, 2)` |

### Functions

| Nix | Javascript |
| --- | --- |
| `(a: { a = a; }) 2` | `(a => ({ a: a }))(2)` |
| `(a: { inherit a; }) 2` | `(a => ({ a }))(2)` |
| `(a: b: { a = a; b = b; }) 2 3` | `((a, b) => ({ a: a, b: b }))(2, 3)` |
| `((a: b: { inherit a b; }) 2) 3` | `(a => b => ({ a, b }))(2)(3)` |
| `({ a }: { inherit a; }) { a = 2; }` | `(({ a }) => ({ a }))({ a: 2 })` |
| `({ a, b, ... }: { inherit a b; }) { a = 2; b = 3; c = 4; }` | `(({ a, b }) => ({ a, b }))({ a: 2, b: 3, c: 4 })` |
| `(a: { c = a.b.c; }) { b = { c = 2; }; }` | `(a => ({ c: a.b.c }))({ b: { c: 2 } })` |
| `(a: { inherit (a.b) c; }) { b = { c = 2; }; }` | `(a => ({ c: a.b.c }))({ b: { c: 2 } })` |
| `({ a ? 2 }: a) {}` | `(({ a = 2 }) => a)({})` |
| `let double = x: x*2; in double 3` | `const double = x => x*2; console.log(double(3));` |
| `let mul = a: (b: a*b); in (mul 2) 3` | `const mul = a => b =>  a*b; console.log(mul(2)(3));` |
| `let x = 3; mul = a: (b: a*b); in (mul 2) 3` | `const x = 3, mul = a => b =>  a*b; console.log(mul(x)(3));` |

### Strings

| Nix | Javascript |
| --- | --- |
| `builtins.stringLength "abc"` | `"abc".length` |
| `builtins.concatStringsSep "\n" ["a" "b" "c"]` | `["a", "b", "c"].join("\n")` |
| `lib.concatMapStringsSep "\n" (s: "${s}x") ["a" "b" "c"]` | ```["a", "b", "c"].map(s => `${s}x`).join("\n")``` |
| `lib.splitString "/" "a/b/c"` | `"a/b/c".split("/")` |
| `lib.flatten (builtins.split "/+" "//a/b")` | `"//a/b".split(/\/+/)` |
| `builtins.match "[a-z]" "a" != null` | `/^[a-z]$/.test("a")` |
| `builtins.elemAt (builtins.match ".*([a-z]).*" "-a-") 0` | `"-a-".match(/^.*([a-z]).*$/)[1]` |
| `lib.flatten (builtins.match "(a) (b)" "a b")` | `"a b".match(/^(a) (b)$/).slice(1)` |
| `builtins.head (builtins.match ".*(b).*" "a b c")` | `"a b c".match(/(b)/)[1]` |
| `builtins.replaceStrings ["a"] ["b"] "aa"` | `"aa".replace(/a/g, "b")` |
| `builtins.fromJSON ''[0, 1, 2]''` | ``JSON.parse(`[0, 1, 2]`)`` |
| `builtins.toJSON [0 1 2]` | `JSON.stringify([0, 1, 2])` |
| `"x=${builtins.toString 1}"` | <code>\`x=${1}\`</code> |
| `builtins.getEnv "HOME"` | `require("process").env["HOME"]` |
| `let stringIndexOf = string: needle: let parts = builtins.split needle string; in if builtins.length parts == 1 then -1 else builtins.stringLength (builtins.elemAt parts 0); in stringIndexOf "abbbc" "b"` | `"abbbc".indexOf("b")` |
| `builtins.substring 2 1 "abcde"` | `"abcde".substr(2, 1)` |
| `builtins.substring 2 (3-2) "abcde"` | `"abcde".slice(2, 3)` |
| `let substringEnd = start: string: let length = builtins.stringLength string; in builtins.substring start length string; in substringEnd 2 "abcde"` | `"abcde".slice(2)` |
| `let stringIndexOfFirstSpace = string: let matches = builtins.match "([^[:space:]]*)([[:space:]]).*" string; in if matches == null then -1 else builtins.stringLength (builtins.elemAt matches 0); in stringIndexOfFirstSpace "a \n\tb c"` | `var m = /\s/.exec("a \n\tb c"); m ? m.index : -1` |
| `lib.escapeShellArg "a\nb"` | <code>require("[shlex](https://www.npmjs.com/package/shlex)").quote("a\nb")</code> |
| `lib.escapeShellArgs ["a" "b c"]` | <code>require("[shlex](https://www.npmjs.com/package/shlex)").join(["a", "b c"])</code> |
| `lib.toUpper "abc"` | `"abc".toUpperCase()` |
| `lib.toLower "ABC"` | `"ABC".toLowerCase()` |

### Arrays

| Nix | Javascript |
| --- | --- |
| `builtins.elemAt [0 1 2] 2` | `[0, 1, 2][2]` |
| `builtins.elem 2 [0 1 2]` | `[0, 1, 2].includes(2)` |
| `builtins.length [0 1 2]` | `[0, 1, 2].length` |
| `builtins.map (x: x+1) [0 1 2]` | `[0, 1, 2].map(x => x+1)` |
| `builtins.filter (x: x>0) [0 1 2]` | `[0, 1, 2].filter(x => x>0)` |
| `[1 2] ++ [3 4]` | `[1, 2].concat([3, 4])` |
| `builtins.concatLists [[1 2] [3 4] [5 6]]` | `[].concat([1, 2], [3, 4], [5, 6])` |
| `builtins.head [0 1 2 3 4]` | `[0, 1, 2, 3, 4][0]` |
| `lib.lists.init [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(0, -1)` |
| `lib.lists.take 4 [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(0, 4)` |
| `lib.lists.last [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(-1)[0]` |
| `builtins.tail [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(1)` |
| `lib.lists.drop 2 [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(2)` |
| `lib.lists.sublist 1 3 [0 1 2 3 4]` | `[0, 1, 2, 3, 4].slice(1, 4)` |
| `builtins.genList (x: x * x) 5` | `Array.from({ length: 5 }).map((_, x) => x * x)` |
| `lib.lists.unique [1 1 2 2 3 3]` | `Array.from(new Set([1, 1, 2, 2, 3, 3]))` |
| `lib.lists.naturalSort ["100" "20" "3"]` | <code>["100" "20" "3"].sort(require("[natsort](https://www.npmjs.com/package/natsort)")())</code> |
| `lib.lists.findFirst (v: v == 2) null [0 1 2]` | `[0, 1, 2].find(v => v == 2) \|\| null` |

### Objects

| Nix | Javascript |
| --- | --- |
| `builtins.attrNames {a=1;}` | `Object.keys({a:1})` |
| `builtins.attrValues {a=1;}` | `Object.values({a:1})` |
| `lib.mapAttrsToList (k: v: [k v]) {a=1;}` | `Object.entries({a:1})` |
| `lib.attrsToList {a=1;}` | `Object.entries({a:1}).map(([k, v]) => ({name: k, value: v}))` |
| `builtins.listToAttrs (map (x: { name = "${x.name}2"; value = x.value + 1; }) (lib.attrsToList {a=1;}))` | ```Object.fromEntries(Object.entries({a:1}).map(([k, v]) => [`${k}2`, v + 1]))``` |
| `builtins.mapAttrs (k: v: v) {a=1;}` | `Object.fromEntries(Object.entries({a:1}).map(([_k, v]) => [_k, v]))` |
| `builtins.listToAttrs [ {name="a"; value=1;} ]` | `Object.fromEntries(["a", 1])` |
| `lib.mapAttrsToList (k: v: v) {a=1;}` | `Object.entries({a:1}).map(([k, v]) => v)` |
| `lib.filterAttrs (k: v: true) {a=1;}` | `Object.fromEntries(Object.entries({a:1}).filter(([k, v]) => true))` |
| `lib.filterAttrs (k: v: (lib.elem k ["a" "b"])) {a=1;c=3;}` | `Object.fromEntries(Object.entries({a:1,c:3}).filter(([k, v]) => ["a", "b"].includes(k)))` |
| `builtins.hasAttr "a" {a=1;}` | `Object.hasOwn({a:1}, "a") == "a" in {a:1}` |
| `{a=1;} ? a` | `Object.hasOwn({a:1}, "a") == "a" in {a:1}` |
| `{a=1;}.a` | `{a:1}.a` |
| `{a=1;}.${"a"}` | `{a:1}["a"]` |
| `builtins.getAttr "a" {a=1;}` | `{a:1}["a"]` |
| `Object.assign({}, {a:1}, {b:2})` | `{a=1;} // {b=2;}` | 

### Files

| Nix | Javascript |
| --- | --- |
| `builtins.baseNameOf "a/b/c.d"` | `require("path").basename("a/b/c.d")` |
| `builtins.dirOf "a/b/c.d"` | `require("path").dirname("a/b/c.d")` |
| `builtins.readDir "/"` | `require("fs").readdirSync("/")` |
| `builtins.readFile "/proc/uptime"` | `require("fs").readFileSync("/proc/uptime", "utf8")` |
| `builtins.fetchurl { url = "https://github.com/"; sha256 = ""; }` | `await fetch("https://github.com/")` |

## Links

- https://github.com/nix-community/awesome-nix#learning
- https://nixos.org/manual/nix/stable/language/operators.html
- https://nixos.org/manual/nix/stable/language/builtins.html
- https://learnxinyminutes.com/docs/nix/
- https://medium.com/@MrJamesFisher/nix-by-example-a0063a1a4c55
- https://nixery.dev/nix-1p.html
- https://nixos.org/manual/nix/unstable/command-ref/new-cli/nix3-eval.html
- https://nixos.wiki/wiki/Nix_command/eval
- https://nixos.org/guides/nix-pills/functions-and-imports.html
- https://ryantm.github.io/nixpkgs/functions/
- https://teu5us.github.io/nix-lib.html
- https://github.com/NixOS/nixpkgs/blob/master/lib/
- https://github.com/NixOS/nixpkgs/blob/master/lib/attrsets.nix

## Expression vs statement

You can read that Nix is Nix expression language.

>Expression: Something which evaluates to a value. Example: 1+2/x
>
>Statement: A line of code which does something. Example: GOTO 100

Or another explanation:

Expression -- from the New Oxford American Dictionary:

> expression: Mathematics a collection of symbols that jointly express a quantity : the expression for the circumference of a circle is 2πr.

Statement from [Wikipedia](http://en.wikipedia.org/wiki/Python_(programming_language)#Statements_and_control_flow):

>In computer programming a statement can be thought of as the smallest standalone element of an imperative programming language. A program is formed by a sequence of one or more statements. A statement will have internal components (e.g., expressions).

- https://stackoverflow.com/questions/19132/expression-versus-statement/19224#19224
- https://stackoverflow.com/questions/4728073/what-is-the-difference-between-an-expression-and-a-statement-in-python/4730559#4730559

### Example

Below `const a` is a statement which consist of expression. [Conditional (ternary) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) is an expression.

```javascript
const a = true ? 1 : 0;
```

## JSON

Description from [Nickel](https://github.com/tweag/nickel), possible successor of Nix language:

>Its purpose is to automate the generation of static configuration files - think JSON, YAML, XML, or your favorite data representation language - that are then fed to another system. It is designed to have a simple, well-understood core: it is in essence JSON with functions.

