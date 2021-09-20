# SERENADE - SEcure *REmote-'N'-Authenticated* DEvTools

**Collaborate on Bugs Using Chrome DevTools remotely over a Secure Proxy from Any Browser**

*Alpha release*

## What is this?

Nothing fancy folks, just a simple 
HTTPS+WebSocket Proxy Server with Auth to 
expose DevTools from a browser on the machine you run it on
so you can work and collaborate, on web apps and bugs, remotely.

Perfect for debugging remotely in collaboration with other humes.
Connect to and debug remote tabs from any device\*.

\* This patched version of Chrome DevTools works in latest Firefox, Safari and Chrome on dekstop and mobile (as tested). Mobile (especially iOS) has bugs.

## Background 

A version of this tool was originally part of the closed-source paid version of my [secure remote browser](https://github.com/i5ik/ViewFinder), but the secure remote debugging capability proved so useful I decided to package it up, copy it out, and make it its own bona-fide open-source product for everyone to use, for free.

I searched around a bit before doing so, and while I couldn't find any current prior art that was up to date in 2021, here was some prior art that I found:

- [auchenberg/devtools-remote](https://github.com/auchenberg/devtools-remote) - An experimental HTTP and WebSocket proxy for DevTools from 2016

## Get it

```sh
$ npm i -g srad
## or
$ npx srad
## or
$ npm i --save srad
## or
$ git clone https://github.com/i5ik/secure-remote-devtools.git
$ cd secure-remote-devtools/
$ npm i
```

## Use it

From the command line:

```sh
srad 9222:mysite.com:8888
```

Using npx:

```sh
npx srad 9222:me.example.com:8080
```

From a NodeJS script:

```javascript
import srad from 'srad';

srad({
  browserPort: 9222,
  serverPort: 8888
}).then(serverStatus => console.log(`Login URL: ${serverStatus.loginUrl}`));
```

From the repository:

```sh
$ cd secure-remote-devtools/
$ npm start 9222:mysrad.int:8555
```

## Security

For security, ***don't expose your browser port (by default 9222) to the public internet***.

This server uses [helmet](https://github.com/helmetjs/helmet), HTTPS, and WSS (*secure WebSockets*).

Once you start `srad` (either via the command line or from the library) you will receive a login URL. That URL can be used to log you on to the secure DevTools server. Without it you will not be able to access any DevTools endpoints. Pass it out to those frens you wish to collaborate with on the solvage, ever venerable, of the buggs.

**srad** uses cookie authentication to prevent unauthorized connections. The need for a secure remote connection utility for DevTools is [well known](https://bugs.chromium.org/p/chromium/issues/detail?id=813540)

## Certificates

By default, srad looks for TLS certificates *(`cert.pem, chain.pem, fullchain.pem  and privkey.pem`)* in `path.resolve(os.homedir(), 'sslcerts')` *(`$HOME/sslcerts` on Windows)*. You can override that with the `certBasePath` option. 

`srad` will ***always*** throw an error and fail is certificates are not found.

## API 

All the options you see below can be accessed via script using their camel-cased variants. Globally installed command line usage (`npm i -g srad@latest`) is shown for demonstrative purposes. The command line API is equivalent whether you use `npx` or `npm start` from the repository to run it.

### Basic Use

The command line has a very simple format:

> srad <BROWSER_PORT>:<DOMAIN_NAME|IP_ADDRESS>:<SERVER_PORT> [certificatesPath]

Where `DOMAIN_NAME|IP_ADDRESS` is that of the server you run `srad` on.

And `certificatesPath` is an optional file system path to override the default location to look for [certificates](#Certificates).

### Browser Port

The port that you have exposed the remote debugging protocol on, via the `--remote-debugging-port` Chrome command line argument. Simple the first positional argument after the command. I.e, say your browser is on port 51386, you'd start a server that is running remote DevTools with. For exmaple, to get up and running with chrome headless, make sure you have chrome installed, then try the following:

```sh

$ google-chrome-stable --headless --remote-debugging-port=51386 srad 51386:mysite.example.com:8080

{
  sradUp: {
    at: 2021-09-20T12:39:24.942Z,
    CHROME_PORT: 51386,
    SERVER_PORT: 8080,
    loginUrl: 'https://mysite.example.com:8080/login?token=a24a30ea17c71f6500b963b732cb2b69fb8d853f'
  }
}

```

There's no default, you must always specify a browser port. If the browser is not running on that port, `srad` will thrown an error. 

### Background

Run it in the background, like so:

```sh
$ srad 51386:doppelgange.pointbyne.org:8888 &
```

Or using pm2:

```sh
$ pm2 start srad 9222:example.spacedemons.com:8433
```

## Server Port

There's no default port, so you must always specify a server port.

### Technical Details and Limitations

By default the Chrome DevTools Frontend does not work cross-browser. It only works in Chrome. This is not a policy of the DevTools team, simply because they don't have the bandwidth to support this right now. 
I [opened a PR to bring cross-browser support to DevTools](https://github.com/ChromeDevTools/devtools-frontend/pull/165), but until and unless we make it happen, I am having `srad` patch the DevTools frontend resources *in-flight* via the proxy, so it can work in any browser.

Because of this ad-hoc, "off-branch" solution, and given the fact that each version of Chrome may ship with a slightly different version of the DevTools front-end, you may find it breaks at any time.

### Other disclaimers

This project has zero association, endorsement or any relationship with with Google, Chrome, the Chrome Dev team, Chrome DevTools front-end or any of the authors.

