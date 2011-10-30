mockgyver
=========

mockgyver is an Erlang tool which will make it easier
to write EUnit tests which need to replace or alter
(stub/mock) the behaviour of other modules.

mockgyver aims to make that process as easy as possible
with a readable and concise syntax.

mockgyver is built around two main constructs:
**?WHEN** which makes it possible to alter the
behaviour of a function and another set of macros (like
**?WAS\_CALLED**) which check that a function was called
with a chosen set of arguments.

Read more about constructs and syntax in the
documentation for the mockgyver module.

A quick tutorial
----------------

Let's assume we want to make sure a fictional program
sets up an ssh connection correctly (in order to test
the part of our program which calls ssh:connect/3)
without having to start an ssh server.  Then we can use
the ?WHEN macro to replace the original ssh module and
let connect/3 return a bogus ssh\_connection\_ref():

    ?WHEN(ssh:connect(_Host, _Port, _Opts) -> {ok, ssh_ref}),

Also, let's mock close/1 while we're at it to make sure
it won't crash on the bogus ssh\_ref:

    ?WHEN(ssh:close(_ConnRef) -> ok),

When testing our program, we want to make sure it calls
the ssh module with the correct arguments so we'll add
these lines:

    ?WAS_CALLED(ssh:connect({127,0,0,1}, 2022, [])),
    ?WAS_CALLED(ssh:close(ssh_ref)),

For all of this to work, the test needs to be
encapsulated within the ?MOCK macro, but there are
helpers that make that process easy.

The final test case could look something like this:

    ?WHEN(ssh:connect(_Host, _Port, _Opts) -> {ok, ssh_ref}),
    ?WHEN(ssh:close(_ConnRef) -> ok),
    ...start the program and trigger the ssh connection to open...
    ?WAS_CALLED(ssh:connect({127,0,0,1}, 2022, [])),
    ...trigger the ssh connection to close...
    ?WAS_CALLED(ssh:close(ssh_ref)),

History
-------

It all started when a friend of mine (Tomas
Abrahamsson) and I (Klas Johansson) wrote a tool we
called the stubber.  Using it looked something like this:

    stubber:run_with_replaced_modules(
        [{math, pi, fun() -> 4 end},
         {some_module, some_function, fun(X, Y) -> ... end},
         ...],
        fun() ->
            code which should be run with replacements above
        end),

Time went by and we had test cases which needed a more
intricate behaviour and that's when I thought: it must
be possible to come up with something better and that's
when I wrote mockgyver.

Building
--------

Build mockgyver using [rebar][1].  Also, [parse\_trans][2]
is required, but rebar takes care of that.

```sh
$ git clone git://github.com/klajo/mockgyver.git
$ rebar get-deps
$ rebar compile
'''


[1]: https://github.com/basho/rebar
[2]: https://github.com/esl/parse_trans
