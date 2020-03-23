# Tor Denial of Service Defenses for Onion Services
Author: [Valentin Franck] <gaspar_ilom@campus.tu-berlin.de>

This is just a prototype and should not be used in production, mostly because it was not written with security in mind, but only to proof and evaluate the concept.
Currently, the challenges, that clients resolve to obtain tokens, are not yet integrated. In theory, a client would have to solve a hard challenge, before it obtains tokens.

## Prerequisites:
The same as in vanilla Tor.
Cryptography is written for OpenSSL 1.1.1.

## Build
To build with the defenses enabled checkout the branch `dos-defenses-master` and proceed as usual:

    git checkout dos-defenses-master
    sh autogen.sh && ./configure && make && make install

## Enable the Defenses for an Onion Service
To enable the defenses for an onion service add the following lines to your torrc:

    # set the HSDir as in vanilla Tor
    HiddenServiceDir /path/to/hs-dir

    # ...configure the onion service as in vanilla Tor.

    # only v3 onion services are supported
    HiddenServiceVersion 3

    # enables the defenses
    HiddenServiceEnableHsDoSDefense 1

    # number of tokens that may be requested per challenge.
    # note, there are no challenges implemented!
    HiddenServiceEnableHsDoSDefenseTokenNum 100
    
    # set the maximum rate for INTRODUCE2 cells without tokens.
    # exceeding cells will be discarded, unless they contain a token.
    HiddenServiceEnableHsDoSRatePerSec 10000

    # set the maximum burst for INTRODUCE2 cells without tokens.
    # exceeding cells will be discarded, unless they contain a token.
    HiddenServiceEnableHsDoSBurstPerSec 10000

## Enable the Defenses for a Client
To enable the defenses for a client add the following lines to your torrc:

    # enable the client to use the defenses, for all services that support it
    HsDoSRetrieveTokens 1
    
    # use this directory to store retrieved tokens for future redemptions.
    # if not set, tokens are only kept in memory by the tor process.
    HsDoSClientDir /path/to/client-dir 

## How do the Defenses work?
The DoS defenses are token-based. Service operators define rate limits for introductions. Once these rate limits are exceeded clients may only connect to the onion service, if they include a valid token in their introduction cells. Tokens are only issued by the onion service after the client solved a challenge.

Tokens are based on verifiable oblivious pseudo random functions (V-OPRFs). The cryptographic scheme is borrowed from Privacy Pass: https://blog.cloudflare.com/privacy-pass-the-math/

## Evaluation
There is a branch `dos-defenses-master-evaluation` which by default prints some benchmarks to the CLI, when the DoS defenses are used by either client or onion service.
Furthermore, to evaluate the cryptographic operations, the test `test_hs_dos_token_benchmark` can be run.

## Tests
There are some unit tests for the cryptographic operations and for encoding and decoding of service descriptors. The rest of the code is mostly untested.

## Further Information
This prototype has been implemented as part of a master thesis, which can be found under the branches `dos-defenses-master` and `dos-defenses-master-evaluation` in the file: `thesis.pdf`