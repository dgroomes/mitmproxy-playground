# mitmproxy-playground

📚 Learning and exploring mitmproxy, the HTTP proxy.

> An interactive TLS-capable intercepting HTTP proxy for penetration testers and software developers.
> 
> -- <cite>https://github.com/mitmproxy/mitmproxy</cite>

**NOTE**: This project is designed for my own personal use.


## Overview

My software development workflow is enriched when I have access to excellent tools. A proxy like mitmproxy looks like it can really help me develop and debug HTTP-based applications. I'm particularly interested in mitmproxy because it is open source, well documented, and seemingly well maintained and focused in its feature set. Let's learn it. 

This project demonstrates how to use mitmproxy as an explicit HTTPS proxy so that we can intercept and control the outbound requests of our app-under-development.

I'm only interested in explicit proxying, not transparent proxying. In explicit proxying, we tell our client software "hey tool, use this proxy server". `curl`, for example, has a `--proxy` flag. This explicitness works for me and is overall simpler to implement and understand. As an aside, the clarity of concepts and narrative in mitmproxy's documentation on explicit vs. transparent proxying is pristine. See the [*How mitmproxy works*](https://docs.mitmproxy.org/stable/concepts/how-mitmproxy-works) page. I have a lot of confidence in this tool. 


## Instructions

Follow these instructions to set up mitmproxy and intercept HTTP traffic between `curl` and various hosts.

1. Make an HTTP request without a proxy
   * To baseline our understanding, let's make a curl request without the proxy. Use the following command.
   * ```shell
     curl --head https://www.w3.org
     ```
   * The response looks something like the following.
   * 
     ```text
     HTTP/2 200 
     content-type: text/html; charset=utf-8
     ```
2. Install mitmproxy via Homebrew
   * ```shell
     brew install --cask mitmproxy
     ```
3. Start the proxy server
   * ```shell
     mitmdump --listen-host 127.0.0.1 --flow-detail 3
     ```
   * You should see output like the following, indicating that the mitmproxy server is running.
   * ```text
     HTTP(S) proxy listening at 127.0.0.1:8080.
     ```
   * The first time mitmproxy is run, it generates a CA certificate in a conventional location. Notice that it created CA certificates in `~/.mitmproxy`.
4. Make a request through the proxy
   * In a new shell session, run a curl command that's like the earlier one, but adorned with extra flags to point curl to the proxy server and the certificate.
   * ```shell
     curl --head --proxy http://127.0.0.1:8080 --cacert ~/.mitmproxy/mitmproxy-ca-cert.pem https://www.w3.org
     ```
   * The curl response should be similar to before, but this time, mitmproxy has intercepted the request and response. Switch back to the mitmproxy terminal to see the traffic. It should look like the following.
   * ```text
     client connect
     server connect www.w3.org:443:443)
     user-agent: curl/8.7.1
     accept: */*
     
     << HTTP/2.0 200 OK 0b
     content-type: text/html; charset=UTF-8
     
     client disconnect
     server disconnect www.w3.org:443
     ```
   * Because we wired up all the certificates correctly, the request and response header content is visible and not encrypted. Very neat.
5. Block traffic
   * Now we'll configure mitmproxy to actually shape traffic by blocking a specific host. We can do this with the `block_list` option and a [filter expression][filter-expressions]. Stop the current mitmproxy instance (Ctrl+C) and restart it with the following command. 
   * ```shell
     mitmdump --listen-host 127.0.0.1 --flow-detail 3 --set block_list=":~d mozilla\\.org:403"
     ```
   * The value of the `block_list` option here is dense. It took me a while to understand it. It is a filter expression. The leading `:` declares our preferred character to use as the delimiter. This should remind you of `sed`'s substitute command where you might write `s/old/new/` but you could just as well write `s:old:new:`. The `~d` is an *operator* for domain names. The `mozilla\\.org` is a regular expression. The following `:` is a delimiter, and the 403 says that we want requests matching this filter to be met with a 403 response. 
6. Test the filtering proxy
   * Repeat the request to `w3.org` again. It should still work. Then, try a request to the blocked domain. It should be rejected with a 403.
   * ```shell
     curl --head --proxy http://127.0.0.1:8080 --cacert ~/.mitmproxy/mitmproxy-ca-cert.pem https://mozilla.org
     ```


## Wish List

General clean-ups, TODOs and things I wish to implement for this subproject:

* [x] DONE Filtering rules to only block traffic to a domain, like mozilla.org.
* [ ] SKIP (Tried it, don't like it for a demo) Consider using `mitmproxy` for interactive use, instead of `mitmdump`. That way, we should be able to add the block rule interactively and that would make for a better demo.


## Reference

* [mitmproxy docs: *Options*][options]
* [mitmproxy docs: *Filter expressions*][filter-expressions]


[options]: https://docs.mitmproxy.org/stable/concepts/options/
[filter-expressions]: https://docs.mitmproxy.org/stable/concepts/filters/
