---
layout: page
title: Portfolio
---

There is not much code that I wrote, that I am allowed to share. But here is a little tool for calculating the numbers
for the german tax for my options trading with Tasty Works.

---

## Tastyworks Tax Calculator

**A command-line tool for calculating German taxes from Tastyworks option trades**

[![GitHub](https://img.shields.io/badge/GitHub-elch78%2Ftastyworks--tax--calculator-blue?logo=github)](https://github.com/elch78/tastyworks-tax-calculator)

### Overview

This tool processes Tastyworks transaction CSV files and calculates gains and losses from option trades, specifically designed for German tax reporting requirements. It handles currency conversions using historical exchange rates from the European Central Bank.

### Key Features

- Processes Tastyworks transaction CSV exports
- Calculates gains and losses for option trades
- Currency exchange rate conversion using ECB historical data
- Command-line interface for easy integration into workflows

### Technical Stack

- **Language:** Kotlin
- **Build System:** Gradle
- **Testing:** Gherkin for behavior-driven testing
- **Data Processing:** CSV parsing and currency exchange rate handling

### Usage

1. Download Tastyworks transactions as CSV
2. Download ECB exchange rates
3. Run application with transaction directory path

### Important Note

This tool is provided for informational purposes only and does not constitute tax advice. Calculations are made to the best of our knowledge, but no guarantee is given for the accuracy, completeness, or currentness of the information provided. Users should consult a tax professional for definitive tax guidance.

---

*More projects coming soon as they become publicly available.*
