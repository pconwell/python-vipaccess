# python-vipaccess
This is a fork of [@bdwyertech](https://github.com/bdwyertech/python-vipaccess)'s work, which in turn is a fork of [@cyrozap](https://github.com/cyrozap)'s [`python-vipaccess`](https://github.com/dlenski/python-vipaccess) original project.

Main differences between this project and [@bdwyertech](https://github.com/bdwyertech/python-vipaccess)'s fork:

- striped out everything that is not strictly necessary (including docker, CI, manifests, etc).
- [pipenv](https://docs.pipenv.org/) instead of setup.py.

## Intro
Symantec's VIP Access client is a fairly standard OTP generator, except that Symantec locks you in to using their app. This script genereates the ID and secret key necessary to add a Symantec supported service (such as ebay) to a generic OTP app such as Authy or Google Authenticator (or any of the other hundreds of OTP managmenet apps).

## Dependencies
-  [`lxml`](https://pypi.python.org/pypi/lxml/3.4.0)
-  [`oath`](https://pypi.python.org/pypi/oath/1.2)
-  [`pycryptodome`](https://pypi.python.org/pypi/pycryptodome/3.4.7)
-  [`requests`](https://pypi.python.org/pypi/requests/)


## Install
You can install the dependencies manually, but if you are using pipenv you can follow the below steps which will take care of everything for you.

1. Install pipenv (if you have not already)
2. `$ git clone git@github.com:pconwell/python-vipaccess.git`
3. `$ cd python-vipaccess`
4. `$ pipenv install`
5. `$ pipenv shell`
6. `$ python cli.py provision -p`


## Usage
### General Usage
```
usage:  cli.py [-h] [-p] {provision,show} ... [-o ./tokens/file.txt] [-t TOKEN_MODEL]

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

### Print ID/key pair
```
$ pipenv shell
$ python cli.py provision -p
```

which will output something similar to:
```
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

### Save ID/key pair to file
`$ python cli.py provision -o ./tokens/example.txt`

This will generate a file at /tokens/example.txt. Rename the output file as you see fit.

Example file output:
```
version 1
secret ULOGV232CEXDHOBFZHHC56WD4M45XXAY
id VSST96853489
expiry 2021-07-29T14:48:33.889Z
```

### Generate OTP codes from saved ID/key file
```
$ python cli.py show -f ./tokens/example.txt
756607
```

