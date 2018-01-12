# Minimist.re

A no-frills cli argument parser for reason, inspired by [minimist](https://www.npmjs.com/package/minimist) and [yargs](https://www.npmjs.com/package/yargs).

## Example usage
(from [pack.re](https://www.npmjs.com/package/pack.re)):
```reason

let parse = Minimist.parse(~alias=[("h", "help")], ~presence=["help"], ~multi=["rename"], ~strings=["base"]);

let help = {|
# pack.re - a simple js bundler for reason

Usage: pack.re [options] entry-file.js > bundle.js

  --base (default: current directory)
      expected to contain node_modules
  --rename newName=realName (can be defined multiple times)
      maps `require("newName")` to a node_module called "realName"
  -h, --help
      print this help
|};

let fail = (msg) => {
  print_endline(msg);
  print_endline(help);
  exit(1);
};

switch (parse(List.tl(Array.to_list(Sys.argv)))) {
| Minimist.Error(err) => fail(Minimist.report(err))
| Ok(opts) =>
if (Minimist.StrSet.mem("help", opts.presence)) {
  print_endline(help); exit(0);
} else switch (opts.rest) {
  | [] => fail("Expected entry file as final argument")
  | [entry] => process(
      ~base=?Minimist.get(opts.strings, "base"),
      ~renames=
        List.map(item => switch (Str.split(Str.regexp("="), item)) {
        | [alias, m] => (alias, m)
        | _ => fail("Expected rename argument to be of the form alias=realname")
        }, Minimist.multi(opts.multi, "rename")),
      entry
    ) |> print_endline
  | _ => fail("Only one entry file allowed")
}
};
```
