# Finding template
When writing findings ensure they are coincise, well written and understandable. Dividing findings in sections helps enhancing report clarity.
Here is a template written in MarkDown language:

```md
[CRITICAL/HIGH/MEDIUM/LOW/INFO] Finding title

- **Location(s):** myFile.sol#1

- **Description:** Within `myFile.sol` the function `myFunc()` is supposed to ...

    ```solidity
    function add(uint256 x, uint256 y) public returns (uint256) {
        return x + y;
    }
    ```
    However if X happens then Y will happen causing Z.

- **Recommendation:**  Consider changing ... in order to ...

- **Status:**  Unresolved.

- **Updates:**
    - `[Client_Name, DD/MM/YYYY]`: The client acknowledged the issue and stated: "..." / The client resolved the issue in commit ....
    - `[Your_Name, DD/MM/YYYY]`: ...


```