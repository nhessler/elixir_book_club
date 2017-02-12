# Elixir in Action - Chapter 1

#### Thoughts after reading this chapter

Reading this chapter, I want to learn more about Elixir.
Among other amazing things, I was intrigued that Erlang and Elixir alone can play
many different roles in server systems.
As a newbie in web development, I could take advantage of that multi-tasking
nature of Erlang and Elixir for me to deepen my understanding of web-related technologies.

\- Masatoshi Nishiguchi

---

## About Erlang
A development platform that allows us to build constantly-available, reliable and responsive system.

#### Some benefits of using Erlang
- fault tolerance
- scalability
- distribution
- responsiveness
- live code update

#### Erlang virtual machine (BEAM)
- parallelizes execution of Erlang programs as much as possible
- uses its own schedulers

#### Erlang processes
- share no memory
- communicate through asynchronous messages
- per-process short garbage collection

#### Erlang in server-side system
- Erlang alone can fill in for various popular technologies:
  + HTTP server (like Nginx and Phusion Passenger)
  + Request processing (like Ruby on Rails)
  + Long-running requests (like Go)
  + Server-wide state (like Redis)
  + Persistable data (like Redis and MongoDB)
  + Background jobs (like Cron, Bash and Ruby)
  + Service crash recovery (like Upstart)
- Entire system can be a single BEAM instance
- Easier to manage

#### OTP (Open Telecom Platform)
- Official distribution: Erlang/OTP
- general-purpose framework that abstracts away many typical Erlang tasks:
  + compiling Erlang
  + starting a BEAM instance
  + creating deployable releases
  + running the interactive shell
  + connecting to the running BEAM instance
  + etc
- cross-platform(*NIX, WINDOWS, etc)
- open-source

---

## About Elixir
An alternative language for the Erlang virtual machine, which allows us to write
cleaner and more compact code. Due to its support and smart compiler architecture,
most of Elixir is written in Elixir.

#### Some benefits of using Elixir (vs Erlang)
- more readable
- less boilerplate
- macro - Elixir code that runs at compile time
- makes functional programming easier with small, reusable and composable functions
