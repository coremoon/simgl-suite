
# simGL

**simGL** (*Sim*plicity *GL*oryfied) is a Python-like dialect designed to simplify the creation of Bitcoin-based smart contracts built on **SimplicityHL**.
It is primarily aimed at **FinTech and finance professionals** who want to develop Bitcoin financial contracts without delving into low-level blockchain programming.

---

## Motivation

* **SimplicityHL** is a higher-level language for Bitcoin smart contracts in the Rust style ([simplicity-lang.org](https://simplicity-lang.org/)).
* Several features crucial for financial contracts are missing:

  * Easy currency and amount handling
  * Clear, intuitive control structures
  * Automatic type inference

**simGL** addresses these gaps by offering:

* **Python-like syntax** for readability and accessibility
* Automatic type assignment
* Simple and familiar control structures like `if/else` and ternary expressions

---

## Example

```python
def main():
    a: u8 = 1
    b: u8 = 2
    # ternary operator similar to Python
    c: u8 = a if a < b else b
    pass
```

* `u8` denotes an 8-bit unsigned integer, as in SimplicityHL
* Syntax is designed to be **intuitive**, allowing the developer to focus on the financial logic of the contract

---

## Advantages

* **Easy onboarding for finance professionals** – no deep Rust knowledge required
* **Strong type safety** with optional automatic type inference
* **Secure and deterministic smart contracts** for Bitcoin / Liquid
* Supports common financial logic: timelocks, covenants, owner & permission checks

---

## Target Audience

* FinTech developers
* Smart contract designers for Bitcoin or Liquid
* Developers who want to focus on **financial logic** rather than low-level blockchain details

---


# Currency Types

simGL introduces **native currency types** to make financial smart contracts **safer and more intuitive**.

## Syntax

Currency types follow the pattern:
```
sys.currency.<scale-unit>.<unit>
```
(with abbreviation `sys.cy` accepted)

where `<scale-unit>` is an optional SI prefix and `<unit>` is the currency unit (e.g., `BTC`, `USD`).

## Scale Units

The following SI prefixes are supported:

| Prefix | Name | Factor |
|--------|------|--------|
| `G` | Giga | 10⁹ |
| `M` | Mega | 10⁶ |
| `k` | Kilo | 10³ |
| — | — | 1 |
| `m` | Milli | 10⁻³ |
| `μ` | Micro | 10⁻⁶ |
| `n` | Nano | 10⁻⁹ |
| `p` | Pico | 10⁻¹² |

## Common Abbreviations

- `sys.currency.BTC` → Bitcoin in full BTC units
- `sys.currency.m.BTC` → Bitcoin in milli-BTC (1/1000 BTC)
- `sys.currency.k.BTC` → Bitcoin in kilo-BTC (1000 BTC)
- `sys.currency.USDT` → Tether (USD stablecoin anchored 1:1 to USD, converted to Bitcoin via bridge services)
- `sys.currency.EUR` → EUR (fiat reference or EUR-backed stablecoin)
- `sys.currency.interest` → Unitless interest rates or multipliers
- `sys.currency.bp` → Basis points - standard interest rate quoting scaled by 1/10000

## Features

Currency types automatically handle **unit conversions**. When multiplying amounts with different scales, the compiler produces the correct output type.

**Benefits:**

- Prevents accidental unit mistakes (BTC vs mBTC)
- Makes financial logic explicit and readable
- Supports safe arithmetic and type checking at compile time

## Conversion Rates

To combine currencies in arithmetic operations, you must declare the conversion rate first.

### Example 1: Cross-Currency (Fiat)

To add USD and EUR amounts:
```python
amount_u: sys.cy.USD = 1.0
amount_e: sys.cy.EUR = 2.0
```

Direct addition fails at compile time because no conversion rate is defined:
```python
amount_total: sys.cy.USD = amount_u + amount_e  # ❌ Compilation error: missing conversion rate
```

Declare the conversion rate using the quoted rate (1 USD = 1.1624 EUR, as of 2026-01-16):
```python
sys.cy.convert(sys.cy.USD, sys.cy.EUR, 1.1624)  # Define: 1 USD = 1.1624 EUR

amount_u: sys.cy.USD = 1.0
amount_e: sys.cy.EUR = 2.0
amount_total: sys.cy.USD = amount_u + amount_e  # ✅ Now valid
```

The compiler converts EUR amounts to USD using the defined rate:

$$
\text{amount\_total} = 1.0 + \frac{2.0}{1.1624} \approx 2.72 \text{ USD}
$$

### Example 2: Crypto to Stablecoin

To add BTC and USDT amounts:
```python
amount_btc: sys.cy.BTC = 1.0
amount_usdt: sys.cy.USDT = 50000.0
```

Declare the conversion rate (1 BTC = 97,500 USDT, as of 2026-01-16):
```python
sys.cy.convert(sys.cy.BTC, sys.cy.USDT, 97500.0)  # Define: 1 BTC = 97,500 USDT

amount_btc: sys.cy.BTC = 1.0
amount_usdt: sys.cy.USDT = 50000.0
amount_total: sys.cy.BTC = amount_btc + amount_usdt  # ✅ Now valid
```

The compiler converts USDT amounts to BTC using the defined rate:

$$
\text{amount\_total} = 1.0 + \frac{50000.0}{97500.0} \approx 1.51 \text{ BTC}
$$

---

## Example: Simple Interest Calculation
```python
# Define amounts with explicit currency types
principal: sys.currency.BTC = 1.0           # 1 BTC principal
interest_rate: sys.currency.interest = 0.05 # 5% interest factor (unitless)

# Automatic conversion: result in milli-BTC
interest_earned: sys.currency.m.BTC = principal * interest_rate

# Output: interest_earned being  50 mBTC (1 BTC × 0.05 = 0.05 BTC = 50 mBTC)
```

**Explanation:**

1. `principal` is defined in full BTC units
2. `interest_rate` is a unitless multiplier (5%)
3. Multiplying them produces `0.05 BTC`
4. simGL automatically converts this to `50 mBTC` to match the declared type
5. The developer doesn't need to manually handle unit conversions
```python
# Define amounts with explicit currency types
principal: sys.currency.BTC = 1.0           # 1 BTC principal
interest_rate: sys.currency.interest = 0.05 # 5% interest factor (unitless)

# Automatic conversion: result in milli-BTC
interest_earned: sys.currency.m.BTC = principal * interest_rate

print(interest_earned)  # Output: 50 mBTC (1 BTC × 0.05 = 0.05 BTC = 50 mBTC)
```

**Explanation:**

1. `principal` is defined in full BTC units
2. `interest_rate` is a unitless multiplier (5%)
3. Multiplying them produces `0.05 BTC`
4. simGL automatically converts this to `50 mBTC` to match the declared type
5. The developer doesn't need to manually handle unit conversions




## Example: Last Will Covenant

A **Last Will** is a recursive covenant that enables multi-signature inheritance scenarios with timelocks. It demonstrates how simGL handles complex financial logic with multiple spend paths and state management.

The contract protects Bitcoin UTXOs through three distinct authorization paths:

1. **Inheritor Path** – After 180 days of inactivity, the designated inheritor can claim the funds (inheritance scenario)
2. **Cold Key Path** – The owner can immediately break out of the covenant using a cold key (secure backup recovery)
3. **Hot Key Path** – The owner can move coins while maintaining the covenant by using a hot key (active management)

The covenant enforces that when spending with the hot key, the first output must reproduce the same script (recursive covenant), and a fee output is required in the second output. This ensures the covenant persists across transactions while preventing unauthorized dispersal.
```python
# LAST WILL
#
# The inheritor can spend the coins if the owner doesn't move them for 180
# days. The owner has to repeat the covenant when he moves the coins with his
# hot key. The owner can break out of the covenant with his cold key.

# ---------------------------------------------------------------------------
# CHECKSIGNATURE
# ---------------------------------------------------------------------------
def checksig(pk, sig):
    """
    Verify that the provided signature matches the given public key for the
    current transaction hash.

    Raises AssertionError if verification fails.
    """
    msg = jet.sig_all_hash()
    # implicit failure in simplicityHL → explicit in Python
    assert jet.bip_0340_verify((pk, msg), sig)


# ---------------------------------------------------------------------------
# RECURSIVE COVENANT
# ---------------------------------------------------------------------------
def recursive_covenant():
    """
    Enforces the covenant in the first output, requires a fee output in the
    second output, and disallows further outputs.
    """
    # Must have exactly 2 outputs
    assert jet.num_outputs() == 2

    this_script_hash = jet.current_script_hash()

    # First output script hash must match current script
    output_script_hash = jet.output_script_hash(0)
    assert output_script_hash != None
    assert this_script_hash == output_script_hash

    # Second output must be a fee output
    assert jet.output_is_fee(1)


# ---------------------------------------------------------------------------
# KEY CONVENTIONS
# ---------------------------------------------------------------------------
# The public keys used here are deterministic multiples of secp256k1 generator G:
#
#   1 * G  -> inheritor key
#   2 * G  -> owner's cold key
#   3 * G  -> owner's hot key
#
# These hex values are x-only BIP-340 public keys for demonstration purposes.
# They are NOT secure and must not be used in production.
# ---------------------------------------------------------------------------

def inherit_spend(inheritor_sig):
    """
    Inheritor spend path: allowed after 180-day relative timelock.
    Signature must match 1*G (inheritor key).
    """
    days_180 = 25920
    # Enforce relative timelock
    assert jet.check_lock_distance(days_180)

    inheritor_pk = (
        0x79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798
    )  # 1 * G

    checksig(inheritor_pk, inheritor_sig)


def cold_spend(cold_sig):
    """
    Owner Cold Key spend path: no timelock.
    Signature must match 2*G (cold key).
    Immediately breaks the covenant.
    """
    cold_pk = (
        0xc6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5
    )  # 2 * G

    checksig(cold_pk, cold_sig)


def refresh_spend(hot_sig):
    """
    Owner Hot Key spend path: signature must match 3*G (hot key).
    Recursive covenant enforced in the first output.
    Allows owner to move coins while continuing the covenant.
    """
    hot_pk = (
        0xf9308a019258c31049344f85f89d5229b531c845836f99b08601f113bce036f9
    )  # 3 * G

    checksig(hot_pk, hot_sig)

    # Enforce covenant repetition
    recursive_covenant()


# ---------------------------------------------------------------------------
# MAIN ROUTINE
# ---------------------------------------------------------------------------
def main():
    """
    Entry point of the script. Selects exactly one authorized spend path
    based on the witness data.

    Witness format:
    {
        "PATH": "inheritor" | "cold" | "hot",
        "SIG": <signature>
    }

    Any invalid path triggers script failure.
    """
    # Safe witness access
    path = witness.PATH
    sig = witness.SIG

    if path == "inheritor":
        inherit_spend(sig)
    elif path == "cold":
        cold_spend(sig)
    elif path == "hot":
        refresh_spend(sig)
    else:
        panic()
```

**Key Concepts Demonstrated:**

- **Multi-Path Authorization:** Different keys control different spend conditions
- **Timelocks:** The 180-day relative timelock enforces the inheritance delay
- **Recursive Covenants:** The hot key path reproduces the covenant to maintain state across transactions
- **Fee Management:** The second output enforces proper transaction fee handling
- **Key Hierarchy:** Demonstrating the distinction between hot keys (active use) and cold keys (secure backup)