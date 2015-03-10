# Codiad-LDAPExternalAuth
LDAP External Authentication Drop-In for Codiad

Written by Korynkai of QuantuMatriX Technologies.

Author's notes:
As this is more of a configuration / authentication drop-in, I see no
reason to create a separate license file.

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Installation

* Download `ldap.php`.
* Edit `ldap.php` in a text editor, changing configuration values as needed.
* Save `ldap.php` somewhere on the server, preferably somewhere within the Codiad root (I created a special directory for External Authentication called `auth` on my setup) and ensure the server has permissions to at least read the file.
* Edit Codiad's `config.php` in a text editor, uncommenting and/or adding the line `define("AUTH_PATH", "/path/to/ldap.php");`. Replace "/path/to" with the actual path. You may use the `BASE_PATH` directive if you saved `ldap.php` to somewhere within the Codiad root. For example, on my setup (with the `auth` directory), this is set to `define("AUTH_PATH", BASE_PATH . "/auth/ldap.php");`
* (Optionally) back up your Codiad `data/users.php` file to somewhere safe (in case of an LDAP issue), then edit `data/users.php` so every field marked `"password"` looks like `"password":null`. This is probably unnecessary, but ensures that no Codiad internal passwords are used and authentication occurs strictly over LDAP.

## Configuration

Most of the configuration should be completed during the installation; However it would be wise to explain these values:

* `$server` would be your LDAP server's connection URI; For example:
 * `$server = 'ldap://ldap.example.com:389';`

* `$basedn` would be your LDAP server's search base distinguished name. This would be where Codiad looks for user entries within LDAP. Example:
 * `$basedn = 'ou=people,dc=example,dc=com';`

* `$filter` is your LDAP user search filter. This tells Codiad which attribute/value pairs to look for as the username to look up. If you aren't sure what to do here, you may use one of the alternatives or use the references either at http://tools.ietf.org/search/rfc4515 (quite technical IETF RFC) or http://goo.gl/FOdGp7 (CentOS documentation page on LDAP search filters). The variable `$1` must always be supplied as a value as it signifies the username. The default will allow a CN or an email to log in; however, the user environments between the CN and email logins would differ, essentially acting as separate users within Codiad. Examples:
 * `$filter = '(&(objectClass=*)(|(cn=$1)(email=$1)))';` <-- Allows CN or email to denote the username. As it uses a logical `or` (`|`), it allows more than one field to directly act as the username, in effect allowing each LDAP user (with both a CN and an email attribute) to create/log-in to two Codiad users if they so desire.
 * `$filter = '(&(objectClass=*)(cn=$1))';` <-- Strictly use CN as username.
 * `$filter = '(&(objectClass=*)(email=$1))';` <-- Strictly use email as username.
 * `$filter = '(&(objectClass=*)(uniqueIdentifier=$1))';` <-- Strictly use uniqueIdentifier as username (This is useful for custom self-identifiable usernames; This is the filter we use on our setup, however it requires additional configuration on LDAP.

* `$createuser` either allows or denies the automatic creation of a Codiad user upon successful LDAP authentication. If set to true, a `user` will be created if the user successfully authenticates through LDAP but is not present within Codiad's `data/users.php` file. If set to `false`, the user will be denied access if they are not present within Codiad's `data/users.php` file, regardless of whether or not the user has successfully authenticated to LDAP. Default is `true`.

* `$version` -- **_DO NOT CHANGE THIS!!! I SERIOUSLY CANNOT STRESS THIS ENOUGH!!!_** The version parameter is the LDAP **_PROTOCOL_** version used by the server and should not be changed unless you are **_ABSOLUTELY, POSITIVELY, 100% CERTAIN_** you are using an oddball version of the **_PROTOCOL_**, either a custom variation, a newer version (highly unlikely -- the protocol versions are directly managed by IETF and are rare to come out -- v3 came out in 1997 and officially made v2 obsolete in 2003, to give an idea) or a version predating 2003. --**_DO NOT_** confuse this with the OpenLDAP, ActiveDirectory, 389, etc... _server_ version. This is **_not_** the same thing.

