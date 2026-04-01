# costpush

**costpush** is a Stata command that implements the cost-push model to estimate the **direct and indirect price effects** of a cost shock (e.g., a tax or price change) using an input-output framework.

Developed by **Paul Corral** — World Bank Group, Equity Policy Lab.

---

## Background

The cost-push model traces how a price or tax shock propagates through an economy via inter-industry linkages. A shock to one sector raises costs not only in that sector (direct effect) but also in all sectors that use it as an input (indirect effect). This command operationalizes the methodology described in:

> CEQ Chapter 7 (June 2017): *Indirect Effects of Fiscal Policy* — included in this repository as `CEQ_Ch7_June2017v2 (002).pdf`.

---

## Files

| File | Description |
|------|-------------|
| `costpush.ado` | Stata command implementing the cost-push model |
| `costpush.sthlp` | Stata help file |
| `CEQ_Ch7_June2017v2 (002).pdf` | Methodological reference (CEQ Chapter 7) |

---

## Installation

Copy `costpush.ado` and `costpush.sthlp` to your Stata `ado` directory, or to your working directory. You can find your personal ado path by running:

```stata
sysdir
```

**Requirements:** Stata version 11.2 or higher.

---

## Syntax

```stata
costpush varlist [if] [in], ///
    fixed(varname)          ///
    priceshock(varname)     ///
    genptot(newvarname)     ///
    genpind(newvarname)     ///
    [fix]
```

### Required Arguments

| Argument | Description |
|----------|-------------|
| `varlist` | A set of numeric variables representing the **input-output (Leontief) coefficient matrix** — must be square (rows = cols) |
| `fixed(varname)` | Variable indicating which sectors are **directly affected** by the shock (1 = fixed/shocked sector, 0 = otherwise) |
| `priceshock(varname)` | Variable containing the **magnitude of the price or tax shock** for each sector |
| `genptot(newvarname)` | Name of the new variable to store the **total price effect** (direct + indirect) |
| `genpind(newvarname)` | Name of the new variable to store the **indirect price effect** only |

### Options

| Option | Description |
|--------|-------------|
| `fix` | If specified, restricts the indirect shock propagation to non-fixed sectors only |

---

## How It Works

The command uses Mata for matrix operations. Given:

- **A**: the input-output coefficient matrix
- **fixed**: a diagonal selection matrix for shocked sectors
- **dp**: the price shock vector

It computes:

1. **Indirect shock** (`dptilda`): the propagated price effect through upstream linkages, via the Leontief inverse `[I - A]⁻¹`.
2. **Total shock** (`deltap`): the combined direct and indirect effect on final prices.

The core matrix formula (from `_indeff` in the Mata block):

```
alfa  = I - diag(fixed)
k     = [I - alfa' * A]⁻¹
dptilda = (dp * A) * k          (restricted to non-fixed sectors if fix is specified)
deltap  = dp * diag(fixed) + (dptilda + dp')' * alfa
```

---

## Example

```stata
* Suppose you have a 3x3 input-output table stored as variables a1 a2 a3,
* with fixed = 1 for the energy sector (row 1), and a 10% price shock.

costpush a1 a2 a3,          ///
    fixed(energy_fixed)     ///
    priceshock(dp)          ///
    genptot(total_effect)   ///
    genpind(indirect_effect)

* View results
list total_effect indirect_effect
```

---

## Output Variables

| Variable | Label | Description |
|----------|-------|-------------|
| `genptot` | Total shock | Combined direct + indirect price effect per sector |
| `genpind` | Indirect shock | Price effect propagated through input-output linkages only |

---

## Reference

Lustig, N. (Ed.). *Commitment to Equity Handbook: Estimating the Impact of Fiscal Policy on Inequality and Poverty*. Chapter 7: Indirect Effects. CEQ Institute, Tulane University. June 2017.

---

## Author

**Paul Corral**  
World Bank Group — Equity Policy Lab  
GitHub: [pcorralrodas](https://github.com/pcorralrodas)

---

## License

This project is made available for research and policy use. Please cite the CEQ methodology and the author when using this command in published work.
