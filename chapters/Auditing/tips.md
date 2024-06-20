# Auditing tips
A list of useful tips when approaching an audit:
1. Change your mindset from "I need to check the validity of what the developer did" to "I am an attacker that wants to break this protocol and (possibly) gain profit".
2. Carefully read docs and Readme if available.
3. Go through the main contracts with all the function bodies closed and build a mental model of the project in your head.
4. Break the system down in pieces: classify components that handle assets, components that have the most impact on the system and so on...
5. List down entities and roles that interact with the system.
6. Start diving into main contracts and main functions with the function bodies opened. In a first pass consider external call safe for now and assume that calls to other functions in the contract are computing everything correctly and return a correct value to the function you are looking at now. In later passes dive into the called functions as well.
7.  Read function comments if available.
8.  Don't trust any line of the codebase. Assume everything is breachable.
9.  Don't trust developer's math knowledge. Assume they are bad in math and double check every math operation.
10. Assume that logic invariants are weak and can be bypassed.
11. Take notes on the codebase with tags or comments for everything that comes to your mind even in the earlier stages of the audit. Save them for a later check when you will better understand the codebase. See tools section for more.
12. Examine contracts not as standalone entities: contract interacts with each others and may have more complex and cross-contract vulnerabilities.
13. Check EVERY external call for reentrancy, DoS, gas griefing. Assume that each external call will callback your contract. How the attacker can gain advantage of this?
14. Always check token decimals compatibility
15. Write PoC for avoiding false positives.   