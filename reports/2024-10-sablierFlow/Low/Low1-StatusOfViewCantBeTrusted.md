
## Vulnerability Details

`statusOf()` suffers from the same issue described in [previous Cantina audit](https://cdn.cantina.xyz/reports/cantina_sablier_october2024.pdf) at issue ***3.3.6***.

The issue, generally explained is a **view** function that is sandwiched by state changes to make on-chain integrators read a different state than the actual one. 

It is not a known issue, this is because it was known that this issue affected the `depletionTimeOf()` function, yet there is nothing said about this also affecting the `statusOf()` function.

Thus, even though the attack vector is of the same nature, it affects to an unknown part of the code.

## Impact

Same impact as described in Cantina report. In this case the `statusOf()` function can be sandwiched by the `sender` of the stream, making a `PAUSED` stream to look `STREAMING` or vice versa:

To make a `STREAMING` stream look `PAUSED`:

```text
// Front-run -> `pause()`
 
// Third party viewer -> `statusOf()`

// Back-run -> `restart()`
```

## Recommendations

Same recommendation described in the previous report:

```text
It should explicitly be made clear to integrating protocols 

that they should not rely on this function, and its only purpose

is to support off chain frontends without any risk to being manipulated.
```
