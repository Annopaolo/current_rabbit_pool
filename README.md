# `ex_rabbitmq_pool`

A RabbitMQ connection pooling library written in Elixir

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `bugs_bunny` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:ex_rabbitmq_pool, "~> 0.1.0"}
  ]
end
```

[![Coverage Status](https://coveralls.io/repos/github/esl/ex_rabbitmq_pool/badge.svg?branch=master)](https://coveralls.io/github/esl/ex_rabbitmq_pool?branch=master)
[![Build Status](https://travis-ci.com/esl/ex_rabbitmq_pool.svg?branch=master)](https://travis-ci.com/esl/ex_rabbitmq_pool)
[HexDocs](https://hexdocs.pm/ex_rabbitmq_pool)
[Hex.pm](https://hex.pm/packages/ex_rabbitmq_pool)

## General Overview

- `ex_rabbitmq_pool` creates a pool of connections to RabbitMQ
- each connection worker traps exits and links the connection process to it
- each connection worker creates a pool of channels and links them to it
- when a client checks out a channel out of the pool the connection worker monitors that client to return the channel into it in case of a crash


## High Level Architecture

when starting a connection worker we are going to start within it a pool of multiplexed channels to RabbitMQ and store them in its state (we can move this later to ets). Then, inside the connection worker we are going to trap exits and link each channel to it. this way if a channel crashes, the connection worker is going to be able to start another channel and if a connection to RabbitMQ crashes we are going to be able to restart that connection, remove all crashed channels and then restart them with a new connection; also we are going to be able to easily monitor client accessing channels, queue an dequeue channels from the pool in order to make them accessible by 1 client at a time making them race condition free.