## Deduction 🧠

### I've uploaded this document before the ending date of the audit thus, so as to no-one copies my finding, this document has limited information. To contextualize this math prodecure you should read all the text in my finding first. ❗❗❗

> 📘 **Note** ℹ️ As all numbers are positive, multiplicating doesnt change inequality symbol, for now.

**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t - d \cdot \frac{s}{C} \cdot e_c \cdot t \geq \frac{s}{O} \cdot e_O \cdot t + \frac{d \cdot e_c \cdot t}{n_O}$

**Classic Keyboard Display:** `(s / C) * e_C * t - d * (s / C) * e_C * t >= ((s / O) * e_O * t) + (d * e_C * t) / n_O`


## Simplification Steps 👣


0️⃣. **Add the fractions on the right side**

**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t - d \cdot \frac{s}{C} \cdot e_c \cdot t \geq \frac{ n_o \cdot s \cdot e_o \cdot t + O \cdot d \cdot e_c \cdot t}{O \cdot n_O}$
    
**Classic Keyboard Display:** `(s / C) * e_C * t - d * (s / C) * e_C * t >= (n_O * s * e_O * t + O * d * e_C * t) / O*n_O`

---

1️⃣. **Factorize \( s \) , \( e_C \) , \( t \) and \( 1/C \) on the left and \( t \) on the right.**

---

**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t \cdot (1 - d)  \geq t \frac{(n_o \cdot s \cdot e_o + O \cdot d \cdot e_c)}{O \cdot n_O}$
    
**Classic Keyboard Display:** `(s / C) * e_C * t *  (1 - d) >= t * ((n_O * s * e_O + O * d * e_C) / O*n_O)`

---

2️⃣. **\( t \) can be cancelled out.**

**LaTeX Display:**
# $\frac{s}{C} \cdot e_C \cdot (1 - d)  \geq \frac{(n_O \cdot s \cdot e_O + O \cdot d \cdot e_c)}{O \cdot n_O}$
    
**Classic Keyboard Display:** `(s/C) * e_C * (1 - d) >= (n_O * s * e_O + O * d * e_C ) / O*n_O`

---

3️⃣. **Multiply both sides by \( C \) and \( O * n_O \)**

**LaTeX Display:**
# $O \cdot n_O \cdot s \cdot e_C \cdot (1 - d)  \geq ( n_O \cdot s \cdot e_O + O \cdot d \cdot e_C ) \cdot C$
    
**Classic Keyboard Display:** `O * n_O * s * e_C * (1-d) >= (n_O * s * e_O + O * d * e_C) * C`

> 🚧 **Note:** ⚠️ The delegation rate \( d \) inherently involves division as \( d = 1/denominatorValue \).

---

4️⃣. **Compute the right side multiplication and then leave all the factors that (s) is multiplying on the right side of the equation and factors without (s) in the left side. Notice we can factor out (n_O) and (s)**

**LaTeX Display:**
# $- O \cdot d \cdot e_C \cdot C \geq (C \cdot e_O - O \cdot e_C \cdot (1 - d)) \cdot s \cdot n_O$
    
**Classic Keyboard Display:** `-O * d * e_C * C >= (C * e_O - O * e_C (1-d)) * s * n_O` 

---

5️⃣. **On the left side of the equation, we have a negative number. This brings up two critical considerations. First, lets divide the factor mulitplying (s) on both sides. Now the sign of the denominator in the left side will determine the direction of the inequality. Second, it's worth noting that this value is no longer guaranteed to remain positive after these manipulations. Therefore, depending on these factors, we are led to two distinct analytical scenarios.**

---

6️⃣. **In the first scenario, where the denominator is positive, the inequality retains its original direction. Consequently, the equation involves dividing a negative value by a positive one, which assures a negative result. This must be greater than or equal to \( s * n_o \), which is definitively positive. Given this contradiction, it becomes clear that the condition for incentives failing is infeasible. As a result, the system's security is guaranteed.**

---

**LaTeX Display:**
# $\frac{-O \cdot d \cdot e_C \cdot C}{(C \cdot e_O - O \cdot e_C \cdot (1 - d))} \geq s \cdot n_O$

**Classic Keyboard Display:** `(-O * d * e_C * C) / (C * e_O - O * e_C * (1-d)) >=  s * n_o` 

7️⃣. **In the second scenario, the denominator is negative. Thus the inequality symbol must be inverted and, when dividing two negative numbers, the result turns positive. Consequently, we're comparing two positive values in the inequality. The condition for proper incentives can either be met or unmet. If it's met, the system's incentive structure remains uncertain as we have proofed that it depends on (s) which is capital from the operator. If unmet, the system's integrity and effectiveness of incentives are assured as we confirm that is false the assumtion that the equation defines which is, checking if comminity pool is more profitable.**

**LaTeX Display:**
# $\frac{-O \cdot d \cdot e_C}{(C \cdot e_O - O \cdot e_C \cdot (1 - d))} \leq s \cdot n_O$
**Classic Keyboard Display:** `(-O * d * e_C * C) / (C * e_O - O * e_C * (1-d)) <=  s * n_o` 

#### Notice that from step 6 we get this other useful condition:

# $(C \cdot e_O - O \cdot e_C \cdot (1 - d)) > 0$
**Classic Keyboard Display:** `C * e_o - O * e_c * (1-d) > 0`