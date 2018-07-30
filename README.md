python-vipaccess
================

This is a fork of [@bdwyertech](https://github.com/bdwyertech/python-vipaccess)'s work, which in turn is a fork of [@cyrozap](https://github.com/cyrozap)'s [`python-vipaccess`](https://github.com/dlenski/python-vipaccess) original project.

Main differences between this project and [@bdwyertech](https://github.com/bdwyertech/python-vipaccess)'s fork:

- striped out everything that is not strictly necessary (including docker, CI, manifests, etc).
- [pipenv](https://docs.pipenv.org/) instead of setup.py.

Intro
-----

python-vipaccess is a free and open source software (FOSS) implementation of Symantec's VIP Access client.

If you need to access a network which uses VIP Access for [two-factor authentication](https://en.wikipedia.org/wiki/Two-factor_authentication), but can't or don't want to use Symantec's proprietary applications—which are only available for Windows, MacOS, Android, iOS—then this is for you. As [@cyrozap](https://github.com/cyrozap) discovered in reverse-engineering the VIP Access protocol
([original blog
post](https://www.cyrozap.com/2014/09/29/reversing-the-symantec-vip-access-provisioning-protocol)),
Symantec VIP Access actually uses a **completely open standard**
called [Time-based One-time Password
Algorithm](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)
for generating the 6-digit codes that it outputs. The only
non-standard part is the **provisioning** protocol used to create a
new token.

Dependencies
------------

-  [`lxml`](https://pypi.python.org/pypi/lxml/3.4.0)
-  [`oath`](https://pypi.python.org/pypi/oath/1.2)
-  [`pycryptodome`](https://pypi.python.org/pypi/pycryptodome/3.4.7)
-  [`requests`](https://pypi.python.org/pypi/requests/)


Install
------

1. Install pipenv (if you have not already)
2. `$ git clone git@github.com:pconwell/python-vipaccess.git`
3. `$ cd python-vipaccess`
3. `$ pipenv install`
5. `$ pipenv shell`
4. `$ python cli.py provision -p`


Usage
-----

The above `$ python cli.py provision -p` command will generate a useable key that looks like this:

```
$ python cli.py provision -p
Generating request...
Fetching provisioning response...
Getting token from response...
Decrypting token...
Checking token...
Credential created successfully:
        otpauth://totp/VIP%20Access:VSST72674373?secret=F7YUK5PD3A6SFCIUFCEJ2G55KLTJL7QS&digits=6&period=30&algorithm=sha1&issuer=Symantec
This credential expires on this date: 2021-07-29T14:46:47.928Z

You will need the ID to register this credential: VSST72674373

You can use oathtool to generate the same OTP codes
as would be produced by the official VIP Access apps:

    oathtool -d6 -b --totp    F7YUK5PD3A6SFCIUFCEJ2G55KLTJL7QS  # 6-digit code
    oathtool -d6 -b --totp -v F7YUK5PD3A6SFCIUFCEJ2G55KLTJL7QS  # ... with extra information
```

The otpauth:// url can be used to generate a qrcode, for example:

```
qrencode -t ANSI256 'otpauth://totp/VIP%20Access:VSST72674373?secret=F7YUK5PD3A6SFCIUFCEJ2G55KLTJL7QS&digits=6&period=30&algorithm=sha1&issuer=Symantec'
```

You can also save the key to a file with `$ python cli.py provision -o ./tokens/example.txt` and it will save a file like this:

```
version 1
secret ULOGV232CEXDHOBFZHHC56WD4M45XXAY
id VSST96853489
expiry 2021-07-29T14:48:33.889Z
```

General Usage:
```
usage: vipaccess provision [-h] [-p | -o DOTFILE] [-t TOKEN_MODEL]

optional arguments:
  -h, --help            show this help message and exit
  -p, --print           Print the new credential, but don't save it to a file
  -o DOTFILE, --dotfile DOTFILE
                        File in which to store the new credential (default
                        ~/.vipaccess
  -t TOKEN_MODEL, --token-model TOKEN_MODEL
                        VIP Access token model. Should be VSST (desktop token,
                        default) or VSMT (mobile token). Some clients only
                        accept one or the other.
```


### Generating access codes using an existing credential

You can also generate OTP codes from saved creditials

```
$ python cli.py show -f ./tokens/example.txt
```

The `vipaccess [show]` option will also do this for you: by default it
generates codes based on the credential in `~/.vipaccess`, but you can
specify an alternative credential file or specify the OATH "token
secret" on the command line.

```
usage: vipaccess show [-h] [-s SECRET | -f DOTFILE]

optional arguments:
  -h, --help            show this help message and exit
  -s SECRET, --secret SECRET
                        Specify the token secret on the command line (base32
                        encoded)
  -f DOTFILE, --dotfile DOTFILE
                        File in which the credential is stored (default
                        ~/.vipaccess
```
