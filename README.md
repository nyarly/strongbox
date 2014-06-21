strongbox
=========

A tool to control private data for deployment and distribution

Design
======

(cuz that's all there is right now)

```
+-- bank - [ gpg blind crypted to all-users ]
|private-metadata
|+-- vault - gpg crypted to user-list-1
||private-metadata
||file1
||file2
|+-----------------------------------
|+-- vault - crypted to user-list-2
||private-metadata
||file3
||file4
|+-----------------------------------
|+-- vault - crypted to user-list-3 ---
||xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
||xxxxx                          xxxxxx
||xxxxx Our user isn't in list-3 xxxxxx
||xxxxx                          xxxxxx
||xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
|+-------------------------------------
+--------------------------------------
```

Banks and vaults are just tars.

Public metadata is encoded as GPG notations.

The bank private metadata includes a bank-wide file directory

Operations
==========
transactions:

0. Copy the bank to a tempfile
0. trap to delete the tempfile
0. operate on the tempfile
0. mv the tempfile to replace the bank
0. emit the new fingerprint of the bank

fingerprint:

0. simple hash of the whole bank

accessable(file?):
0. decrypt each vault
1. list files for the vault
2. collect list of file,vault pairs
3. file queried?
  4. return list.include?(file)
4. else
  5. return list

Everything below here wrapped in a transaction

add(file, [users]):

0. is there an entry for the file already? && die
0. find pubkey for each user || die
1. does a vault with those keys exist?
  0. decrypt that vault
  1. add file
  2. enccypt that vault
2. otherwise
  3. create a new vault
  4. add file
  5. encrypt

remove(file):

0. is there an entry for the file? || die
1. accessable(file)
2. decrypt vault with file
3. remove file
4. encrypt vault

# User interface

Not basic operations, but handy:

add_user(user, file):

0. accessable(file)
1. userlist = vault.users + user
2. remove file
3. add(file, userlist)

remove_user is converse

# More complex

Generate content etag: 
* requires each vault to have an hmac of content as public metadata.
* bank etag is hash of well formed concatenation of hmacs, and is public metadata
* invariant when users added to files

# A Use Case

Application credentials - deploy servers generate a passwordless keypair and publish public key. Add public key to e.g. SSL cert file and transmit strongbox to servers e.g. in git repo or application package.

# Threat models

Worst practical would be an attacker who retained old versions of a strongbox + a compromised key. strongbox would need to be replaced, and not used. One case would be a "break" operation that encrypts the strongbox as a whole to one or two trusted ops who can use it to confirm that all secrets in the new bank are changed.
