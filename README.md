# llmsec
## Problem
All communication from a user to an LLM and back are in plain text. This is a security risk.

Plain text provides attack vectors where both the incoming and the outgoing data can be manipulated. Some of these risks are covered in the [OWASP top 10 GenAI/LLM risks](https://genai.owasp.org/llm-top-10/). Notably, for the context covered here, [prompt injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) and [insecure output handling](https://genai.owasp.org/llmrisk/llm02-insecure-output-handling/). Note that while transport level security, like TLS, alleviates some of these concerns, it does not provide mechanisms to provide enough evidence that the prompt sent and the result returned correlate and have not been tampered with. Proposed [solutions](https://arxiv.org/pdf/2401.07612?) to introduce cryptographic constructs only addresses parts of the problem.

Tp summarize. We have no simple way
* For the LLM to verify it received the prompt the User sent, nor 
* for the User to verify it received what the LLM produced for this prompt.

This proposal addresses this problem. It attempts to be as non-intrusive as possible.

## Proposal

### Setup needed
The User and the LLM share a secret key and agree on how this key is used to sign and verify messages. They also agree on how to hash their messages. 

### Protocol
1. User creates a prompt.

        What is the Matrix?

2. User hashes the prompt `h(prompt)` and makes a note of the hash.
3. User sends message to LLM.
4. LLM receives the message and produces a result.

        Unfortunately, no one can be told what the Matrix is. You have to see it for yourself. 

5. LLM hashes the prompt `hash(prompt)` into `prompt_hash`. 
6. LLM hashes the result `hash(result)` into `result_hash`. 
7. LLM signs the two hashes `sign(key, prompt_hash + result_hash)` and appends this to the result.

        Unfortunately, no one can be told what the Matrix is. You have to see it for yourself. sig:572e11942cd2c09b9477e36431707008

8. LLM sends response back to User.
9. User extracts the signature into `sig` and hashes the result (without the signature) into `result_hash`.  
10. User verifies the signature with the key over the two hashes `verify(key, prompt_hash + result_hash) == sig` (User knows the prompt hash since it made a note of it before sending). 

If verified, the User can be sure that 
* *The LLM saw the same prompt the User sent*, and
* *The User received the response the LLM produced for this prompt*.

### Extended protocol
The protocol can be extended to have User not only hash but sign the prompt, using the same mechanism the LLM uses. This could address [Denial of Service attacks](https://genai.owasp.org/llmrisk/llm04-model-denial-of-service/) concerns.


## Notes
This is a simple writeup. It needs tighter definitions of 
1. [Acceptable](https://en.wikipedia.org/wiki/Internationalization_and_localization) and [proper](https://en.wikipedia.org/wiki/Canonicalization) format of how the signatures are added and parsed in the prompt and response.
2. Formats, and usage of keys. Especially how to agree on hashing and cryptographically signing of messages, likely some type of `HMAC`.