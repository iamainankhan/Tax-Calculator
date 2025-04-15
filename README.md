# Tax-Calculator
import matplotlib.pyplot as plt

class TaxCalculator2025:
    def __init__(self, income, deductions, hra_exemption, capital_gains_stcg, capital_gains_ltcg,
                 business_income, special_deductions, tax_saving_investments, residential_status,
                 is_senior_citizen=False, is_disabled=False):
        self.income = income
        self.deductions = deductions
        self.hra_exemption = hra_exemption
        self.capital_gains_stcg = capital_gains_stcg
        self.capital_gains_ltcg = capital_gains_ltcg
        self.business_income = business_income
        self.special_deductions = special_deductions
        self.tax_saving_investments = tax_saving_investments
        self.residential_status = residential_status
        self.is_senior_citizen = is_senior_citizen
        self.is_disabled = is_disabled

        self.standard_deduction = 50000  # As per Budget 2025
        self.rebate_87A = 25000 if income <= 700000 else 0
        self.cess_rate = 0.04

    def calculate_tax_old_regime(self):
        gross_income = self.income + self.business_income + self.capital_gains_stcg + self.capital_gains_ltcg
        total_deductions = self.deductions + self.tax_saving_investments + self.hra_exemption + \
                           self.special_deductions + self.standard_deduction
        taxable_income = max(gross_income - total_deductions, 0)

        tax = 0
        slabs = [(250000, 0.05), (500000, 0.1), (1000000, 0.2)]

        remaining = taxable_income
        if self.is_senior_citizen:
            remaining -= 300000
        else:
            remaining -= 250000

        for limit, rate in slabs:
            if remaining <= 0:
                break
            taxed_amount = min(remaining, limit)
            tax += taxed_amount * rate
            remaining -= taxed_amount

        if remaining > 0:
            tax += remaining * 0.3

        tax -= self.rebate_87A
        tax = max(tax, 0)
        tax += tax * self.cess_rate

        return round(tax, 2), taxable_income

    def calculate_tax_new_regime(self):
        gross_income = self.income + self.business_income + self.capital_gains_stcg + self.capital_gains_ltcg
        taxable_income = gross_income

        slabs = [
            (300000, 0.0), (300000, 0.05), (300000, 0.1), (300000, 0.15),
            (300000, 0.2), (300000, 0.25)
        ]

        tax = 0
        remaining = taxable_income
        for limit, rate in slabs:
            if remaining <= 0:
                break
            taxed_amount = min(remaining, limit)
            tax += taxed_amount * rate
            remaining -= taxed_amount

        if remaining > 0:
            tax += remaining * 0.3

        if taxable_income <= 700000:
            tax = 0

        tax += tax * self.cess_rate

        return round(tax, 2), taxable_income

    def compare_tax_regimes(self):
        old_tax, old_taxable = self.calculate_tax_old_regime()
        new_tax, new_taxable = self.calculate_tax_new_regime()

        better = 'Old Regime' if old_tax < new_tax else 'New Regime'
        savings = abs(old_tax - new_tax)

        self.display_graph(old_tax, new_tax)

        return {
            'Old Regime Tax': old_tax,
            'New Regime Tax': new_tax,
            'Better Option': better,
            'Tax Saved': savings,
            'Old Regime Taxable Income': old_taxable,
            'New Regime Taxable Income': new_taxable
        }

    def display_graph(self, old_tax, new_tax):
        labels = ['Old Regime', 'New Regime']
        taxes = [old_tax, new_tax]
        plt.figure(figsize=(6, 4))
        plt.bar(labels, taxes, color=['blue', 'green'])
        plt.title('Tax Comparison: Old vs New Regime')
        plt.ylabel('Tax Payable (INR)')
        plt.grid(axis='y', linestyle='--', alpha=0.7)
        plt.show()

def get_user_input():
    def get_float(prompt):
        try:
            return float(input(prompt))
        except ValueError:
            return 0.0

    income = get_float("Enter Gross Annual Income: ₹")
    deductions = get_float("Enter total deductions under Section 80C, 80D, etc.: ₹")
    hra_exemption = get_float("Enter HRA Exemption (if any): ₹")
    capital_gains_stcg = get_float("Enter Short-Term Capital Gains: ₹")
    capital_gains_ltcg = get_float("Enter Long-Term Capital Gains: ₹")
    business_income = get_float("Enter Business Income (for freelancers/self-employed): ₹")
    special_deductions = get_float("Enter Special Deductions (senior citizens, disabled, etc.): ₹")
    tax_saving_investments = get_float("Enter Investment in PPF, NPS, ELSS, etc.: ₹")
    residential_status = input("Enter Residential Status (Resident/NRI): ")
    is_senior_citizen = input("Are you a Senior Citizen? (yes/no): ").strip().lower() == 'yes'
    is_disabled = input("Are you a Disabled individual? (yes/no): ").strip().lower() == 'yes'

    return TaxCalculator2025(
        income=income,
        deductions=deductions,
        hra_exemption=hra_exemption,
        capital_gains_stcg=capital_gains_stcg,
        capital_gains_ltcg=capital_gains_ltcg,
        business_income=business_income,
        special_deductions=special_deductions,
        tax_saving_investments=tax_saving_investments,
        residential_status=residential_status,
        is_senior_citizen=is_senior_citizen,
        is_disabled=is_disabled
    )


if __name__ == '__main__':
    print("\n----Union Budget 2025 Tax Calculator---- ")
    calculator = get_user_input()
    result = calculator.compare_tax_regimes()

    print("\n===== Tax Summary =====")
    for key, value in result.items():
         if isinstance(value, (int, float)):
             print(f"{key}:₹{value:,.2f}")
         else:
             print(f"{key}:₹{value}")
