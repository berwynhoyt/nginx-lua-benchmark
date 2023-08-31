# Lua benchmark: OpenResty vs NGINX+WSAPI

## Installation

Here are instructions to install [OpenResty](https://openresty.org/en/installation.html) and [LuaRocks](https://luarocks.org/#quick-start).

Next install Apache Benchmark and an fcgi header file ( `ab`, `fcgi_stdio.h`), then the LuaRock `wsapi-fcgi`. These instructions work on Ubuntu:

```shell
sudo apt-get install apache2-utils libfcgi-dev
sudo luarocks install wsapi-fcgi
```

Start your nginx server and fcgi server:

```shell
make start
```

Test that nginx is working:

```shell
$ make test
curl http://localhost:8081/ && echo Success
<p>hello, world</p>
Success
curl http://localhost:8082/ && echo Success
<html><body><p>Hello Wsapi!</p><p>PATH_INFO: /</p><p>SCRIPT_NAME: /run.lua</p></body></html>
Success
```

Run the benchmarks:

```shell
make benchmarks
```

## Results

The benchmark results on a quad-core i7-8565 @1.8GHz are as follows, where 8081 is port serving OpenResty's Lua and 8082 is PUC Lua via FastCGI:

```shell
$ make summary
make benchmark | egrep "^ab|Time taken"
ab -k -c1000 -n50000 -S http://localhost:8081/ 2> >(egrep -v "(Completed|Finished).*requests" 1>&2)
Time taken for tests:   0.425 seconds
ab -k -c1000 -n50000 -S http://localhost:8082/ 2> >(egrep -v "(Completed|Finished).*requests" 1>&2)
Time taken for tests:   3.369 seconds
```

In short, OpenResty's Lua solution is over **8× faster** than PUC Lua via FastCGI.

## Troubleshooting

### Too many open files

If you get this error when you run `make benchmark` then the benchmarking is trying to make more simultaneous requests than your user allows. Check the number of requests your user is allowed as follows:

```shell
$ ulimit -Hn
1048576
$ ulimit -Sn
1024
```

These numbers should be significantly greater than the `-c<connections>` parameter in the `ab` command run by `make benchmark`. If not, see how to increase your open file limit [here](https://www.cyberciti.biz/faq/linux-unix-nginx-too-many-open-files/) or [here for Ubuntu](https://manage.accuwebhosting.com/knowledgebase/3334/How-to-Increase-Open-Files-Limit-in-Ubuntu.html).

