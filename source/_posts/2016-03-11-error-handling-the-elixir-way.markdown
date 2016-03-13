---
layout: post
title: "Error Handling, The Elixir way"
date: 2016-03-11 17:39
comments: true
categories: elixir
---

When you start learning Elixir, the first you'll learn is the culture of `let it crash`.
Which sounds great! It measn that you don't need to handle exceptions and the system heal itself
, it's to easy to fall in to trap of not catching any exception like I did.

<!-- more -->


After your application is ready to go live,
you'll understand that not every thing should crash. Consider this example from
[Elide](https://github.com/slashmili/elide/) project and the [changes](https://github.com/slashmili/elide/pull/8)
I've made to handle exceptions.

Elide project is a url shortener service, that I'm using to experiment with Elixir and Phoenix.
I'm using [hashids](https://hex.pm/packages/hashids) to generate the short urls and later when people
use short urls I use the same library to decode a short url to meaning full data that I have in the app,
all simple simple stuff. However while I was developing, I followed my own understanding of `let it crash`
which was to never handle any unexpected behaviour.

Consider simplified version of getting a short url entity:

```elixir
defmodule Elide.ElinkServer
  alias Elide.{Repo, Elink}

  def get_elink(slug) do
    slug
    |> by_slug
    |> Repo.one
  end

  defp by_slug(my_slug) do
    elink_seq = get_details_by_slug(my_slug)
    from e in Elink,
      where: e.elink_seq == ^elink_seq
  end

  defp get_details_by_slug(my_slug) do
    s = Hashids.new(min_len: 1, salt: salt)
    {:ok, [elink_seq]} = Hashids.decode(s, my_slug)
    elink_seq
  end
end
```

The code looks simple and I thought if someone wants to open an invalid short url,
well I `let it crash`!

As soon as I had the project on the production server I noticed that:

* If any users enters an invalid short url, they'll face an ugly page with error `Server internal error`
* I am getting a lot of false alarms on [honeybadger](http://honeybadger.io) from bots on the internet,
they don't care that I don't handle corner cases, they just hit the web server with invalid urls

To be honest I knew I should handle this case and I postponed it.

So what are the options? How can I handle it in elixir in a decent way

* Returning nil on case of failure. Nah, I certainly don't like this one.
How can I know what was the error in the first place if I only return nil.
* Using `try/catch`, yeah it does the job but doesn't look decent. Generally I don't like the codes that are guarded with exceptions.
If I don't have any option, sure but there should be a better way in Elixir and Erlang.
* Returning a tuple of `{:ok, result}` in case of success and `{:error, reason}` in case of failure.
I've seen this one I was using [File](http://elixir-lang.org/docs/stable/elixir/File.html) module.
Looks like that is convention that people are following in Erlang and Elixir.

The idea behind using using tuple as opposed to using `try/catch` is we shouldn't use errors as control flow
([read more](http://elixir-lang.org/getting-started/try-catch-and-rescue.html)).
If a short url doesn't exist in database because a bot requested a random url should be considered as
an invalid data and treated as invalid data. Let's how can we refactor this code to handle invalid data in Elixir way:

```elixir
defmodule Elide.ElinkServer
  alias Elide.{Repo, Elink}

  def get_elink(slug) do
    slug
    |> by_slug
    |> fetch_one
  end

  defp fetch_one({:error, _} = error), do: error
  defp fetch_one({:ok, query}) = {:ok, Repo.one(query)}

  defp by_slug(my_slug) do
    case get_details_by_slug(my_slug) do
      {:error, _} = error -> error
      {:ok, elink_seq} ->
        {:ok, from e in Elink,
          where: e.elink_seq == ^elink_seq}
  end

  defp get_details_by_slug(my_slug) do
    s = Hashids.new(min_len: 1, salt: salt)
    case Hashids.decode(s, my_slug) do
        {:ok, [elink_seq]} -> {:ok, elink_seq}
        _ -> {:error, :failed_to_decode}
    end
  end
end
```

Does it worth it? I believe so with this approach I can simply pattern match the return value and keep the `flow` going.
For example my controller would look like:

```elixir
  def go(conn, %{"slug" => slug}) do
    slug
    |> ElinkServer.get_elink
    |> redirect_to_url(conn)
  end

  defp redirect_to_url({:error, _}, conn) do
    conn
    |> put_status(404)
    |> render(Elide.ErrorView, "404.html")
    |> halt
  end

  defp redirect_to_url({:ok, elink}, conn) do
    url = elink.urls |> Enum.shuffle |> List.first
    conn
    |> inc_stat(elink)
    |> redirect(external: url.link)
    |> halt
  end
```

I could have just used `try/catch` in `go` function and call it done but I certainly like this code much better!
