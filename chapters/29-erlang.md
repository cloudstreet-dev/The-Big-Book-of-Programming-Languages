# Chapter 29: Erlang - Let It Fail (But Keep Running)

## The Telecoms Language That Conquered the Internet

Erlang was created in 1986 at Ericsson by Joe Armstrong, Robert Virding, and Mike Williams to build telecommunications switching systems. They needed a language for systems that couldn't afford downtime—phone switches that must run 24/7/365 for decades.

The solution they created was radical: lightweight processes, message passing, fault isolation, and the philosophy of "let it crash." Instead of preventing all failures (impossible), Erlang isolates them and recovers automatically.

By 2025, Erlang powers WhatsApp (handling billions of messages), RabbitMQ (message queueing), and countless other systems where reliability matters more than raw performance. It's proof that sometimes the best way to handle errors is to embrace failure.

## The Philosophy

Erlang's design principles:

**Concurrency**: Lightweight processes (millions of them). Message passing, no shared state.

**Fault tolerance**: "Let it crash." Isolate failures, restart components.

**Reliability**: Build systems that never stop. Hot code swapping, supervision trees.

**Distribution**: Run across multiple nodes seamlessly.

**Soft real-time**: Low-latency message passing for telecom requirements.

**Immutability**: No mutable state. No race conditions.

The result is a language built for systems that can't afford to fail.

## What It Looks Like

```erlang
%% Comments start with %

%% Atoms (like symbols)
ok
error
{ok, Value}
{error, Reason}

%% Variables (start with uppercase, single assignment)
X = 42.
Name = "Alice".

%% Pattern matching
{ok, Result} = {ok, 100}.
%% Result is now 100

%% Lists
Numbers = [1, 2, 3, 4, 5].
[Head | Tail] = Numbers.  %% Head = 1, Tail = [2,3,4,5]

%% Tuples
Person = {person, "Alice", 30}.

%% Functions
greet(Name) ->
    io:format("Hello, ~s!~n", [Name]).

%% Pattern matching in functions
factorial(0) -> 1;
factorial(N) when N > 0 -> N * factorial(N - 1).

%% Guards
max(X, Y) when X > Y -> X;
max(_X, Y) -> Y.

%% Case expressions
classify(N) ->
    case N of
        0 -> zero;
        N when N > 0 -> positive;
        N when N < 0 -> negative
    end.

%% List comprehensions
Squares = [X*X || X <- [1,2,3,4,5]].
Evens = [X || X <- [1,2,3,4,5,6], X rem 2 =:= 0].

%% Processes and message passing
Pid = spawn(fun() -> loop() end).

send_message(Pid, Message) ->
    Pid ! Message.

receive_message() ->
    receive
        {hello, Name} ->
            io:format("Hello, ~s!~n", [Name]);
        stop ->
            io:format("Stopping~n"),
            ok
    after 5000 ->
        io:format("Timeout~n")
    end.

%% Recursion for loops
loop() ->
    receive
        {From, Message} ->
            From ! {ok, Message},
            loop();
        stop ->
            ok
    end.
```

Key features:
- **Immutability**: Variables are single-assignment
- **Pattern matching**: Everywhere
- **Lightweight processes**: Actor model
- **Message passing**: No shared state
- **Fault tolerance**: Supervision trees

## The Type System

Erlang is dynamically typed:

```erlang
%% Basic types
42                      %% Integer
3.14                    %% Float
atom                    %% Atom
"string"                %% String (actually a list of integers)
<<"binary">>            %% Binary
true                    %% Boolean (actually atom)
false

%% Lists
[1, 2, 3, 4, 5]
[Head | Tail] = [1, 2, 3]
% Head = 1, Tail = [2, 3]

%% Tuples
{ok, 42}
{error, "Something went wrong"}
{person, "Alice", 30}

%% Maps (introduced in Erlang 17)
Person = #{name => "Alice", age => 30}.
#{name := Name} = Person.  %% Pattern matching

%% Functions (first-class)
F = fun(X) -> X * 2 end.
F(5).  % 10

%% Processes (PIDs)
Pid = self().  %% Current process ID

%% References (unique identifiers)
Ref = make_ref().

%% Type specs (optional, for Dialyzer)
-spec add(integer(), integer()) -> integer().
add(X, Y) -> X + Y.
```

Dynamic typing with optional type specifications for static analysis.

## Processes: The Heart of Erlang

Erlang processes are lightweight and isolated:

```erlang
%% Spawn a process
Pid = spawn(fun() ->
    io:format("Hello from process~n")
end).

%% Send a message
Pid ! {hello, "World"}.

%% Receive a message
receive
    {hello, Name} ->
        io:format("Received hello from ~s~n", [Name])
end.

%% Process with state (actor pattern)
-module(counter).
-export([start/0, increment/1, get_value/1]).

start() ->
    spawn(fun() -> loop(0) end).

loop(Count) ->
    receive
        {increment, From} ->
            From ! Count + 1,
            loop(Count + 1);
        {get_value, From} ->
            From ! Count,
            loop(Count);
        stop ->
            ok
    end.

increment(Pid) ->
    Pid ! {increment, self()},
    receive
        Value -> Value
    end.

get_value(Pid) ->
    Pid ! {get_value, self()},
    receive
        Value -> Value
    end.

%% Linking processes (die together)
Pid = spawn_link(fun() -> work() end).

%% Monitoring (one-way notification)
Ref = monitor(process, Pid),
receive
    {'DOWN', Ref, process, Pid, Reason} ->
        io:format("Process died: ~p~n", [Reason])
end.
```

Processes are cheap—you can spawn millions.

## Supervisors: The Safety Net

Supervisors restart failed processes:

```erlang
-module(my_supervisor).
-behaviour(supervisor).

init([]) ->
    Children = [
        #{
            id => worker1,
            start => {worker, start_link, []},
            restart => permanent,
            shutdown => 5000,
            type => worker
        }
    ],
    {ok, {#{strategy => one_for_one}, Children}}.

%% Restart strategies:
%% - one_for_one: Restart only the failed child
%% - one_for_all: Restart all children
%% - rest_for_one: Restart failed child and those started after it
```

Supervision trees create fault-tolerant systems.

## Pattern Matching: The Core Feature

Pattern matching is pervasive in Erlang:

```erlang
%% Function clauses with pattern matching
handle_message({hello, Name}) ->
    io:format("Hello, ~s!~n", [Name]);
handle_message({goodbye, Name}) ->
    io:format("Goodbye, ~s!~n", [Name]);
handle_message(_) ->
    io:format("Unknown message~n").

%% List patterns
sum([]) -> 0;
sum([Head | Tail]) -> Head + sum(Tail).

%% Tuple patterns
area({circle, Radius}) ->
    math:pi() * Radius * Radius;
area({rectangle, Width, Height}) ->
    Width * Height.

%% Map patterns
process_person(#{name := Name, age := Age}) ->
    io:format("~s is ~p years old~n", [Name, Age]).

%% Guards in patterns
classify(N) when N > 0 -> positive;
classify(0) -> zero;
classify(N) when N < 0 -> negative.

%% Case expressions
Result = case Expression of
    Pattern1 -> Action1;
    Pattern2 -> Action2;
    _ -> DefaultAction
end.
```

Pattern matching makes code declarative and clear.

## OTP: The Secret Weapon

OTP (Open Telecom Platform) is Erlang's killer feature:

```erlang
%% GenServer (generic server behavior)
-module(my_server).
-behaviour(gen_server).

%% Client API
start_link() ->
    gen_server:start_link({local, ?MODULE}, ?MODULE, [], []).

get_count() ->
    gen_server:call(?MODULE, get_count).

increment() ->
    gen_server:cast(?MODULE, increment).

%% Server callbacks
init([]) ->
    {ok, 0}.  %% Initial state

handle_call(get_count, _From, Count) ->
    {reply, Count, Count}.

handle_cast(increment, Count) ->
    {noreply, Count + 1}.

handle_info(_Info, State) ->
    {noreply, State}.

terminate(_Reason, _State) ->
    ok.

code_change(_OldVsn, State, _Extra) ->
    {ok, State}.
```

OTP provides battle-tested abstractions for building reliable systems.

## Distribution: Built-In

Erlang makes distributed systems straightforward:

```erlang
%% Start nodes
erl -name node1@localhost
erl -name node2@localhost

%% Connect nodes
net_adm:ping('node2@localhost').

%% Spawn process on remote node
Pid = spawn('node2@localhost', fun() -> work() end).

%% Send message to remote process
Pid ! {hello, "from node1"}.

%% RPC (Remote Procedure Call)
rpc:call('node2@localhost', Module, Function, Args).

%% Globally registered processes
global:register_name(my_service, Pid).
global:whereis_name(my_service).
```

Distribution is transparent—remote processes work like local ones.

## Hot Code Swapping

Erlang allows code updates without stopping the system:

```erlang
%% Load new version of module
code:load_file(my_module).

%% Processes automatically use new code
%% Old processes continue with old code
%% New calls use new code

%% Supervised code upgrade
sys:suspend(Pid).
sys:change_code(Pid, Module, OldVsn, Extra).
sys:resume(Pid).
```

Telecom systems run for years without stopping—hot swapping enables this.

## What Erlang Is Best For

**Messaging systems**: WhatsApp handles billions of messages with Erlang.

**Message queues**: RabbitMQ is written in Erlang.

**Telecom systems**: Ericsson's original use case. Still relevant.

**Distributed systems**: Built-in distribution and fault tolerance.

**Real-time systems**: Low-latency message passing.

**Systems that can't stop**: Fault tolerance and hot code swapping.

## What Erlang Is Worst For

**CPU-intensive computation**: Not designed for number crunching. Use C, Rust, or Julia.

**Quick scripts**: Too much overhead. Use Python or Bash.

**GUIs**: Possible but awkward. Use web technologies or native languages.

**Systems programming**: Too high-level. Use C or Rust.

**Data science**: No ecosystem. Use Python or R.

## The Ecosystem

**Build tool**: rebar3

**Package manager**: hex.pm (shared with Elixir)

**Web frameworks**: Cowboy (HTTP server), Phoenix (via Elixir)

**Databases**: Mnesia (distributed database built into Erlang)

**Testing**: EUnit, Common Test

**Static analysis**: Dialyzer (catches type errors)

**Notable projects**:
- **RabbitMQ**: Message broker
- **CouchDB**: Document database
- **ejabberd**: XMPP server

The ecosystem is smaller than mainstream languages but high quality.

## The Community

**The Telecom Engineers**: Ericsson veterans. Built phone switches.

**The Messaging Platform Developers**: WhatsApp, RabbitMQ, chat systems.

**The Distributed Systems Engineers**: Build fault-tolerant services.

**The Reliability Obsessed**: "Let it crash" converts.

**The Elixir Users**: Use Erlang's BEAM VM with modern syntax.

### Common Phrases
- "Let it crash"
- "Nine nines of availability" (99.9999999% uptime)
- "Processes are cheap"
- "The BEAM is rock-solid"
- "OTP solves this"
- "Pattern match everything"
- "Just use a supervisor"

## Erlang vs Elixir

Elixir runs on the Erlang VM (BEAM) with modern syntax:

**Erlang**:
- Older syntax (Prolog-inspired)
- Mature, battle-tested
- All OTP libraries
- Telecom heritage

**Elixir**:
- Ruby-inspired syntax
- Modern tooling (Mix)
- Same BEAM, same reliability
- Growing community

Both are excellent. Elixir is Erlang for people who want friendlier syntax.

## WhatsApp: Erlang's Showcase

WhatsApp famously used Erlang to handle:
- 2 billion+ users
- 100 billion+ messages per day
- 50 engineers

Erlang's concurrency model and fault tolerance made this possible.

## Should You Learn Erlang?

**Yes, if**:
- You're building distributed, fault-tolerant systems
- Reliability matters more than raw performance
- You need soft real-time guarantees
- You're interested in the actor model
- You want to understand robust system design

**No, if**:
- You're doing web development (use Elixir or other languages)
- You need CPU-intensive computation
- You want mainstream syntax and tooling
- You're building CRUD apps

**Maybe, if**:
- You're curious about fault tolerance and concurrency
- You want to understand the BEAM VM
- You're considering Elixir (learning Erlang helps)

## The Verdict

Erlang is the language for systems that can't fail. It's not the fastest language. It's not the prettiest syntax. But for reliability, fault tolerance, and handling massive concurrency, it's unmatched.

The "let it crash" philosophy sounds reckless but is actually safer. Instead of trying to prevent all errors (impossible), Erlang isolates failures and recovers automatically. Supervision trees ensure failed components restart. Hot code swapping enables updates without downtime.

In 2025, Erlang isn't trendy. Elixir has captured mindshare with friendlier syntax. But Erlang's BEAM VM remains the foundation. WhatsApp's billions of messages, RabbitMQ's reliable queuing, and decades of telecom infrastructure all rely on Erlang's rock-solid reliability.

Is Erlang for everyone? No. The syntax is unusual. The functional style requires adjustment. The ecosystem is specialized. But for building systems that must stay running, Erlang delivers.

Learn Erlang to understand fault tolerance done right. Appreciate the actor model and supervision trees. See how immutability prevents concurrency bugs. Understand why sometimes the best error handling is letting things fail and restarting them.

Erlang won't replace Python or JavaScript. But when you need nine nines of reliability, when downtime costs millions, when failure isn't an option—Erlang is there, quietly running systems that keep the world connected.

Let it crash. The supervisor will restart it. Your system will keep running. That's the Erlang way.

---

**Next**: [Chapter 30: Nim - Python-like Syntax, C-like Speed](30-nim.md)
