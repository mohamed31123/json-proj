# JSON Data Cleaning

A small practice repository for learning how to inspect, normalize, validate, and clean JSON data using PowerShell.

The project uses sample e-commerce orders to demonstrate common data-quality problems such as inconsistent types, missing values, inconsistent field names, mixed date formats, and malformed item structures.

## Goals

- Read and parse JSON files.
- Inspect object properties and data types.
- Normalize inconsistent fields and structures.
- Detect missing or invalid values.
- Validate cleaned records.
- Export cleaned data without overwriting the original dataset.

## Repository Structure

```text
json-data-cleaning/
├── data/
│   ├── data.json              # Original sample data
│   └── data-cleaned.json      # Generated cleaned output
├── scripts/
│   └── clean-data.ps1         # PowerShell cleaning script
└── README.md
```

> The exact files may differ depending on what you add to the repository. Keep the original input data separate from generated output.

## Requirements

- Windows PowerShell 5.1 or PowerShell 7+
- A valid JSON input file at `data/data.json`

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
cd YOUR-REPOSITORY
```

Replace `YOUR-USERNAME` and `YOUR-REPOSITORY` with your GitHub details.

### 2. Inspect the JSON data

In PowerShell, run:

```powershell
$data = Get-Content .\data\data.json -Raw | ConvertFrom-Json
$data.Count
$data[0] | Get-Member -MemberType NoteProperty
```

- `Get-Content -Raw` reads the whole file as one string.
- `ConvertFrom-Json` parses the JSON into PowerShell objects.
- `Get-Member` lists the properties of the first order.

### 3. Run the cleaning script

If the repository contains `scripts/clean-data.ps1`, run:

```powershell
.\scripts\clean-data.ps1
```

The script should read `data/data.json` and write its result to `data/data-cleaned.json`. Review the script before running it and confirm that it does not overwrite the source file.

## Data Quality Issues Covered

| Issue | Example | Intended handling |
|---|---|---|
| Inconsistent ID types | `1001` vs. `"1001"` | Convert to a consistent numeric type when valid |
| Inconsistent field names | `qty` vs. `qte` | Standardize on `qty` |
| Different item structures | `items` as an object vs. an array | Normalize to an array |
| Mixed price formats | `899`, `"899"`, `"$25"` | Parse only when the format and currency are understood |
| Mixed date formats | `2025-08-20`, `21/08/2025` | Convert to a chosen standard after resolving ambiguity |
| Missing values | `price` or `date` is `null` | Flag for review rather than inventing a value |
| Invalid totals | Negative or mismatched total | Validate against the business rules |

## Recommended Normalization Rules

For this sample project, use the following target schema:

- `id`: integer
- `date`: ISO 8601 date or datetime, using a consistent convention
- `customer`: object with a consistent `name` and `email` structure
- `items`: always an array
- `items[].sku`: string
- `items[].qty`: positive integer
- `items[].price`: non-negative decimal number in a documented currency
- `total`: non-negative decimal number, calculated according to documented discount and shipping rules
- `promo`: a consistent object structure when present

Do not assume all amounts use the same currency unless the dataset confirms it. Do not guess missing customer details, dates, or prices.

## Validate the Output

After generating the cleaned file, parse it again:

```powershell
$cleaned = Get-Content .\data\data-cleaned.json -Raw | ConvertFrom-Json
$cleaned.Count
$cleaned[0] | Get-Member -MemberType NoteProperty
```

You can also verify that each order's `items` property is an array:

```powershell
$cleaned | ForEach-Object {
    [PSCustomObject]@{
        Id = $_.id
        ItemsIsArray = ($null -ne $_.items -and $_.items -is [array])
    }
}
```

Passing `ConvertFrom-Json` confirms that the file is syntactically valid JSON; it does **not** prove that all business rules or values are correct.

## Example

Input:

```json
{
  "id": "1001",
  "customer": {
    "name": "Ali",
    "email": " ali@example.com "
  },
  "items": {
    "sku": "mouse",
    "qty": "3",
    "price": "25"
  }
}
```

Possible normalized output:

```json
{
  "id": 1001,
  "customer": {
    "name": "Ali",
    "email": "ali@example.com"
  },
  "items": [
    {
      "sku": "mouse",
      "qty": 3,
      "price": 25
    }
  ]
}
```

This example assumes the string values contain valid numeric data and that the intended currency is already defined by the project.

## Learning Outcomes

By completing this project, you will practice:

- JSON syntax and nested objects/arrays
- PowerShell pipelines and object inspection
- Type conversion and field normalization
- Missing-value handling
- Data validation and reproducible cleaning
- Git and GitHub project documentation

## Future Improvements

- Add automated validation tests.
- Report invalid records to a separate file.
- Detect duplicate order IDs.
- Standardize dates and currency safely.
- Recalculate totals and discounts using explicit business rules.
- Add a Python or Java implementation for comparison.

## Contributing

Contributions and suggestions are welcome. Please describe the data-quality issue and the rule used to fix it. Avoid committing real customer information or other sensitive data.

## License

Choose a license before sharing or reusing this project publicly. For example, an MIT license can be added by creating a `LICENSE` file.

---

**Note:** This repository is intended for educational practice. Always keep an untouched copy of the source data and review cleaning rules before using them on real datasets.
