# Learning log

Written by me, in my own words. One entry per stage.

For each stage: what I built, what broke, what I decided and why, what I
would do differently, and one concept I can now explain.

## Stage 0 --- Project setup

1. learnt that Nestjs work with different adopters, and it strucuture the project mainly into modulers, controllers and main file
2. learnt that oxc offer alternative to eslint, prettier and many different tools with faster Rust performance
3. learnt that with docker compose volums should point sometimes to data folder to pressist the data
4. learnt that to prevent losing redis data since it store in RAM we use appendonly yes option
5. learnt that pnpm has no ci option just like npm and we should always lock and use prod in production build
6. leant that Nestjs has a set of steps that involve DTO (data contract for type safe), Pipes (run before the request -> transfer the data type and add validation), Interceptors (run before or after the router handler, can be used for stuffs like logging), DI (Dependency Injection, make class ask for what it needs)
7. Nestjs already offer a ready moduler for checking configuration (envoriment variables), we can also acess them by import the moduler

   _(not started)_
