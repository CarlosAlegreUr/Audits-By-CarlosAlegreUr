# Basic Analysis Report for veRWA's audit 🗞️ 
###  by [CarlosAlegreUr](https://github.com/CarlosAlegreUr)

# code4arena Questions ❓

### Approach taken in evaluating the codebase:
I first read all the documents in code4arena and the recommended external lectures. I then analyzed the size of the files and their graph dependencies and relations. I started reading the code functions to understand the overall working and synchronization of the system while writing notes about possible exploit scenarios I could imagine. After grasping the understanding, I revised the notes and deleted the ones that eventually made no sense.

### Anything that you learned from evaluating the codebase:
- Understanding of what voting escrows are.
- Knowledge about how to weight voting power.
- Learning English words like "gauge."
- Insight into how to distribute rewards efficiently thanks to the CRV docs.
- Discovery of a blockchain called CANTO, and an overall view of its consensus mechanism.
- Comprehension of how code4arena works.

### Any comments for the judge to contextualize your findings:
Basically, this was my first time auditing in code4arena, and I did it by auditing a code that had zero familiar concepts for me, all in just 3 days. I was excited and took it as a challenge. I only had time to mainly focus on understanding the code rather than "attacking" it, the lack of proper comments and guidance inside the code made the research long and difficult. Additionally, understanding how to submit findings in code4arena and the nuances of labeling them took up more time.

Despite the challenges, I've been adventurous and tackled this uncharted territory project. Here is my analysis of it overall and its areas of improvement:

> 📘 **Note** ℹ️: All these points and more have been addressed with more detail in my QA-LowRisk report.

---

## Summary 📌

This report assesses the accessibility and security of the given code, focusing on its friendliness to developers and the accuracy and completeness of references. The main findings indicate that the code could improve its quality and become more beginner-friendly. The recommendations are provided to make the code more comprehensible, increase the efficacy of auditing, and improve overall security.

---

## 1. Developer-Friendliness

### a. Issues
- 🚫 **Inaccessibility to Beginners**: The code's complexity and lack of proper explanations limit access to new developers.
- 🚫 **Inconsistent Naming and Terminology**: The use of different terms for similar concepts and vague variable names adds to confusion.
- 🚫 **Lack of Documentation and Comments**: Critical parts of the code lack needed explanations.

### b. Recommendations
- ✅ **Enhance Documentation**: Incorporate thorough explanations of variables, functions, and key concepts within the code.
- ✅ **Clarify and Standardize Terminology**: Adopt consistent language and clarify specific terms like 'gauges.'
- ✅ **Include Descriptive Comments**: Annotate the code with comments that elucidate specific functions and variables.

---

## 2. Links and References

### a. Issues
- 🚫 **Inaccurate Links**: Connections to related projects may not be precise or adequately informative.

### b. Recommendations
- ✅ **Detail Differences Between Projects**: Highlight the distinctions between the linked projects and the code.
- ✅ **Provide Contextual Comments**: Explain within the code how specific changes relate to the contract's functionality.

---

## 3. Considerations for Auditing

### a. Recommendations
- ✅ **Provide Adequate Time for Auditors**: If code is not thoroughly commented or documented, it's advisable to allocate more time for auditors to allow for a comprehensive understanding and analysis.

---

## 4. Specific Concerns in `LendingLedger.sol`

### a. Issues
- 🚫 **Potential Frozen Funds**: Known issue of possible frozen funds that needs to be addressed.

### b. Recommendations
- ✅ **Tackle Frozen Funds in the Future**: Consider flagging any funds that were frozen, allowing governance to manage those frozen funds to prevent potential loss.

---

## Conclusion

The revised analysis underscores the importance of accessibility, precise documentation, and attention to specific vulnerabilities within the code. By adopting the recommended measures, future audits could become more efficient and effective.
