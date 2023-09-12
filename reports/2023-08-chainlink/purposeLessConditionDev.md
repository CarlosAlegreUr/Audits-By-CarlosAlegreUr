## Deduction 🧠

### I've uploaded this document before the ending date of the audit thus, so as to no-one copies my finding, this document has limited information. To contextualize this math prodecure you should read all the text in my finding first. ❗❗❗

> 📘 **Note** ℹ️ As all numbers are positive, multiplicating doesnt change inequality symbol.

**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t - d \cdot \frac{s}{C} \cdot e_c \cdot t \geq \frac{s}{O} \cdot e_O \cdot t + \frac{d \cdot e_c \cdot t}{n_O}$

###### **Classic Keyboard Display:** `(s / C) * e_C * t - d * (s / C) * e_C * t >= ((s / O) * e_O * t) + (d * e_C * t) / n_O`


## Simplification Steps 👣


0. **Add the fractions on the right side**

**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t - d \cdot \frac{s}{C} \cdot e_c \cdot t \geq \frac{ n_o \cdot s \cdot e_o \cdot t + O \cdot d \cdot e_c \cdot t}{O \cdot n_O}$
    
###### **Classic Keyboard Display:** `(s / C) * e_C * t - d * (s / C) * e_C * t >= (n_O * s * e_O * t + O * d * e_C * t) / O*n_O`


1. **Factorize \( s \) , \( e_C \) , \( t \) and \( 1/C \) on the left and \( t \) on the right.**


**LaTeX Display:**
# $\frac{s}{C} \cdot e_c \cdot t \cdot (1 - d)  \geq t \frac{(n_o \cdot s \cdot e_o + O \cdot d \cdot e_c)}{O \cdot n_O}$
    
###### **Classic Keyboard Display:** `(s / C) * e_C * t *  (1 - d) >= t * ((n_O * s * e_O + O * d * e_C) / O*n_O)`

2. **\( t \) can be cancelled out.**

    **LaTeX Display:**
# $\frac{s}{C} \cdot e_C \cdot (1 - d)  \geq \frac{(n_O \cdot s \cdot e_O + O \cdot d \cdot e_c)}{O \cdot n_O}$
    
###### **Classic Keyboard Display:** `(s/C) * e_C * (1 - d) >= (n_O * s * e_O + O * d * e_C ) / O*n_O`

3. **Multiply both sides by \( C \) and \( O * n_O \) to eliminate division and facilitate Solidiy's implementation if desired. And there we got the final ***`Purposeless Inequality`***.**

    **LaTeX Display:**
    # $ O \cdot n_O \cdot s \cdot e_C \cdot (1 - d)  \geq ( n_O \cdot s \cdot e_O + O \cdot d \cdot e_C ) \cdot C $
    
    ###### **Classic Keyboard Display:** `O * n_O * s * e_C * (1-d) >= (n_O * s * e_O + O * d * e_C) * C`
