A secure repository for sensitive data
======================================

This repository allows to handle and share securely sensitive data by
wisely using GNU Privacy Guard (GPG).  It consists of a simple directory
structure and a shell script named `secshare`, which is a simple GPG
wrapper.

Requisites
----------

The helper script is designed to work under the following assumptions:

 1. Your team established a chain of trust:
 
    All members should have signed each other's public keys.  If you don't
    know what this means, you should read some good manual on how to do
    it: I would personally suggest [this guide][futureboy].

 2. Your hard drive is encrypted:

    The helper scripts of this repository will store sensitive information
    as plaintext on your hard drive. This is secure only if your disk is
    encrypted.

 3. No revocation of granted access:

    The script knows nothing about your data, and if someone leaves the
    team you probably want to consider all shared secret as compromised.

 4. No fine-grained selection of access: all team has got access to all
    files.

[futureboy]: http://futureboy.us/pgp.html

Concept
-------

Your sensitive data is stored in the `data` directory.  Note that the
`data` directory is added to the `.gitignore` file and **must never be
commited**.

Files and top-level directories under `data` are treated as distinct
sensitive items, so you can group together in the same top-level directory
any tree of related data.

The `secshare` script encrypts each sensitive item as ascii-armored
gpg-encrypted tarball. Each encrypted file is stored in the
`encrypted_data` directory.

A file named `keys` will list the users which are allowed to access data.
This setting is honored by the `secshare` script during the encryption
phase.

The `keys` file is signed by one member of the team. This ensures
the `keys` file to contain only authorized keys, hence avoiding an
attacker to put her key among the recipients of encryption.

Since encrypted files cannot be read, an index file documenting the
content of each of them is recommended, along with descriptive file names.

Helper script
-------------

The `secshare` script is a simple helper program written in Bourne Shell
(sh). The following commands are supported:

 1. `./secshare init`:

    Initialize the repository unless it wasn't already initialized. This
    simply creates the filesystem structure and initializes the `keys`
    file;

 2. `./secshare close`:

    Encrypt each sensitive item (file or directory) in the `data`
    directory. Put the resulting encrypted files in the `encrypted_data`
    directory.

    Recipients are obtained by reading the `keys` file (see the **Keys
    file format** section for details). The `./secshare close` command
    will fail unless the `keys` file can be verified against a detached
    signature `keys.asc`, and the signature was made by a trusted user.

    (A trusted user is someone whose key was signed by you or another
    user you trust. Again, [RTFM][futureboy]!).

 3. `./secshare sign`:

    Sign the `keys` file with a detached signature. This is a required
    step in order for you and other team members to `./secshare close`
    successfully.  One should never sign blindly the `keys` file!

 4. `./secshare open`:

    Open one or more files contained in `encrypted_data`.

    Files cannot be opened unless they were encrypted with the user's
    public key (via `./secshare close`).

    A pattern can be specified to ask for one or more files to be
    decrypted. If not specified, the script will try to decrypt all
    encrypted files.

File format of the `keys` file:
-------------------------------

This file contains references to all keys which will be used as
recipients. Each line starting with the `pub:` prefix should identify a
public key.

What follows the `pub:` prefix will be cut of white-space characters and
used as parameter for `--recipient` in GPG. In other words the short id of
the key or an email address will work fine, but the full key fingerprint
is recommended.

Besides the `pub:` lines, this file can contain arbitrary text, which will
be ignored by the `secshare` script. This "feature" can be used to put
useful comments, as the name of the key owner.

If no key is provided in the keys file GPG will not be provided with
`--recipient` directives, and you will be interactively asked to provide a
recipient.
