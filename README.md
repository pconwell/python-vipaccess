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

## Provisioning (Generating) a new key
### Print ID/key pair
> `provision -p` will generate a key and print it to the terminal. No data is saved.

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
> `provision -o /path/file` will generate a key and save it to the path specified.

`$ python cli.py provision -o ./tokens/example.txt`

This will generate a file at /tokens/example.txt. Rename the output file as you see fit.

Example file output:
```
version 1
secret ULOGV232CEXDHOBFZHHC56WD4M45XXAY
id VSST96853489
expiry 2021-07-29T14:48:33.889Z
```

### Specify Standard or Mobile key format
> `provision -t [VSST|VSMT]` will generate a standard or mobile key format.

Some sites may require a standard (VSST) or mobile (VSMT) format key. The only difference between the formats is the token id will either start with `VSST##########` or `VSMT##########`, otherwise the tokens are identical. If no option is specified, the token will default to standard (VSST) format.


## Generate OTP code
### OTP codes from existing key file
> `show -f /path/file` will generate an OTP for the specified key file.

```
$ python cli.py show -f ./tokens/example.txt
756607
```

### OTP codes from existing secret
> `show -s SECRETKEYSTRINGHERE` will generate an OTP for the specified secret key.

```
$ python cli.py show -s ULOGV232CEXDHOBFZHHC56WD4M45XXAY
756607
```

## Add Secret to Authenicator App
One of the more likely use cases for this project is to generate a standard secret that can be added to any OTP app (Google Authenticator, Authy, etc) instead of being locked in to Symantec's app/token.

The fastest and arguably most secure way is to just run `python cli.py provision -p` then use the provided `ID` and `secret` (it will be the long string of characters on the line that says 'oathtool'...). The `ID` will most likely be entered into whatever website or service you are setting up, and the `secret` will be added to your authenticator app.

The 'safest' way would be to save the token to a file so you can access it later if needed. However, note that if you save the file, you must keep the file safe to prevent someone from stealing the information in the file. The file itself contains everything someone needs to know to steal your OTP codes.
