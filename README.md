# proxy-cert-settings
[Recommended to see this doc in wide preview.](https://github.com/eholic/proxy-cert-settings/blob/main/README.md)

Many enterprise IT provides a proxy server and a local Certificate Authority (CA) for security and inspection.

In that environment, usually tools (e.g. package manager and http library) couldn't pass proxy and fail to verify SSL certificate and raise errors like:

- Proxy Error
    - W: Failed to fetch
    - Proxy Authentication Required
- SSL Error
    - Certificate verification failed
    - [SSL: CERTIFICATE_VERIFY_FAILED]
    - SSLError
    - SSLCertVerificationError
    - SSL certificate problem
    - error self signed certificate in certificate chain

This memorandum is the proxy and CA setting list of some tools.
> Some settings are redundant due to their dependency.

- [Proxy setting](https://github.com/eholic/proxy-cert-settings#proxy-setting)
- [CA setting](https://github.com/eholic/proxy-cert-settings#ca-setting)

Give me PR if you know other settings.

## Proxy setting

Many tools refer to the following environment variables.
```bash
# Some tools may needs upper case variables.
$ export http_proxy=`your_proxy_server`
$ export https_proxy=`your_proxy_server`
$ export no_proxy=localhost,127.0.0.1,::1,*.local
```


|tool | Setting | Reference |
|-|-|-|
|apt|\$ sudo vi /etc/apt/apt.conf.d/80proxy <br />Acquire::http::proxy `your_proxy_server`<br />Acquire::https::proxy `your_proxy_server`|[ubuntu](https://manpages.ubuntu.com/manpages/trusty/man5/apt.conf.5.html)|
|yum|\$ sudo vi /etc/yum.conf <br /> proxy=`your_proxy_server`|-|
|docker|\$ vi \~/.docker/config.json<br /> { "proxies": { "default": {<br /> "httpProxy": `"your_proxy_server"`,<br /> "httpsProxy": `"your_proxy_server"`,<br /> "noProxy": "localhost,127.0.0.1,::1,\*.local"<br /> } } } <br />\$ sudo service docker restart|[docker](https://docs.docker.com/network/proxy/#configure-the-docker-client)|
|git|\$git config --global http.proxy `your_proxy_server`<br />\$git config --global https.proxy `your_proxy_server`|[git-scm](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy)|
|gradle|\$ vi \~/.gradle/gradle.properties<br />systemProp.http.proxyHost=`your_proxy_server`<br />systemProp.http.proxyPort=`port`<br />systemProp.https.proxyHost=`your_proxy_server`<br />systemProp.https.proxyPort=`port`|[gradle](https://docs.gradle.org/current/userguide/build_environment.html#sec:accessing_the_web_via_a_proxy)|


## CA setting

Many tools refer to the following environment variables.
```bash
$ export REQUESTS_CA_BUNDLE=`/your/cafile.crt`
$ export SSL_CERT_FILE=`/your/cafile.crt`
$ export NODE_EXTRA_CA_CERTS=`/your/cafile.crt`
```

|tool |Set ca file (Recommended)| Ignore SSL (Depricated) | Reference |
|-|-|-|-|
|apt| \$ sudo cp `/your/cafile.crt` /usr/local/share/ca-certificates/ <br /> \$ sudo update-ca-certificates <br />|-|[ubuntu](https://ubuntu.com/server/docs/security-trust-store)|
|curl|a) export CURL_CA_BUNDLE=`/your/cafile.crt` <br /> b) or use `--cacert` option |use `-k/--insecure` option|[curl](https://curl.se/docs/sslcerts.html)|
|git|git config --global http.sslCAInfo `/your/cafile.crt`|git config --global http.sslVerify false| [git-scm](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpsslCAInfo)|
|Python|-|-|-|
|pip |a) pip config set global.cert `/your/cafile.crt`<br />b) or use `--cert` option <br /> c) or set env: `PIP_CERT`, `REQUESTS_CA_BUNDLE`, or `CURL_CA_BUNDLE`| a) pip config set global.trusted-host pypi.org\ pypi.python.org\ files.pythonhosted.org <br /> b) or use `--trusted-host` option|[pypa](https://pip.pypa.io/en/stable/cli/pip_install/?highlight=SSL%20Certificate%20Verification#ssl-certificate-verification)|
|urllib|ctx = ssl.SSLContext()<br />ctx.load_verify_locations('`/your/cafile.crt`')<br />u = urllib.request.urlopen('url', context=ctx)|ctx = ssl.SSLContext()<br />ctx.verify_mode = ssl.CERT_NONE<br />u = urllib.request.urlopen('url', context=ctx)|[python](https://docs.python.org/3/library/ssl.html#ssl.SSLContext.load_verify_locations)|
|conda|conda config --set ssl_verify `/your/cafile.crt`|conda config --set ssl_verify False|[conda](https://docs.conda.io/projects/conda/en/latest/user-guide/configuration/use-condarc.html#ssl-verification)|
|-|-|-|-|
|Ruby / gem|copy `cafile.crt` to /path/to/ruby{ver}/lib/ruby/{ver}/rubygems/ssl_certs|\$ gem sources --add http://rubygems.org<br />\$ gem sources --remove https://rubygems.org|[rubygems](https://guides.rubygems.org/command-reference/#gem-sources)|
|Node|-|-|-|
|Node.js|export NODE_EXTRA_CA_CERTS=`/your/cafile.crt`|export NODE_TLS_REJECT_UNAUTHORIZED=0| [nodejs](https://nodejs.org/api/cli.html#node_extra_ca_certsfile)|
|npm|npm config set cafile `/your/cafile.crt`|npm config set strict-ssl false|[npmjs](https://docs.npmjs.com/cli/v8/using-npm/config#cafile)|
|yarn|yarn config set cafile `/your/cafile.crt`|yarn config set strict-ssl false ||
|-|-|-|-|
|AWS CLI| a) aws configure set ca_bundle <br /> b) or use `--ca-bundle` option <br /> c) or set env: `AWS_CA_BUNDLE` | use `--no-verify-ssl` option |[amazon](https://docs.aws.amazon.com/cli/latest/reference/index.html#options)|
