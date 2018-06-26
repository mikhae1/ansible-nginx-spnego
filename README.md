## NGINX with Kereberos Authentication
This role builds nginx from [official sources](https://nginx.org/download)
with [SPNEGO module](https://github.com/stnoonan/spnego-http-auth-nginx-module.git)

#### Kerberos setup
- obtain keytab file and place it at `/etc/krb5.keytab`
- check HTTP principals: `klist -k`
- add grants for nginx user to read keytab file
- setup NGINX

#### NGINX setup
```
upstream sso {
  server 127.0.0.1:{{ sso_port }};
}

location ~ ^/ {
  location ~ ^/check_krb {
    auth_gss on;
    auth_gss_realm {{ krb_realm }};
    # make sure nginx user have acceess to this file
    auth_gss_keytab /etc/krb5.keytab;
    # check HTTP principals with: klist -k
    auth_gss_service_name HTTP/{{ krb_domain }};
    auth_gss_allow_basic_fallback off;

    # Successful SPNEGO authentication

    # X-Remote-User header is set after successful spnego auth
    # and is accessible from upstream
    proxy_set_header X-Remote-User $remote_user@{{ krb_realm }};

    # proceed to upstream
    proxy_pass http://sso;
  }

  proxy_pass http://sso;
}
```
