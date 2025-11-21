#!/usr/bin/env python3

"""
Simple Phytochemistry Calculator

Functions:
1. Extraction yield (%)
2. DPPH radical scavenging (%)
3. Total phenolic content (TPC, mg GAE/g sample)
4. Total flavonoid content (TFC, mg QE/g sample)
"""

def extraction_yield(mass_extract_g: float, mass_sample_g: float) -> float:
    """
    Extraction yield (%) = (mass of dried extract / mass of initial sample) * 100
    """
    if mass_sample_g == 0:
        raise ValueError("Mass of sample cannot be zero.")
    return (mass_extract_g / mass_sample_g) * 100.0


def dpph_scavenging(a_control: float, a_sample: float) -> float:
    """
    DPPH radical scavenging (%) = [(A_control - A_sample) / A_control] * 100
    """
    if a_control == 0:
        raise ValueError("A_control cannot be zero.")
    return ((a_control - a_sample) / a_control) * 100.0


def tpc_mg_gae_per_g_sample(absorbance: float,
                            slope: float,
                            intercept: float,
                            dilution_factor: float,
                            extract_conc_mg_per_ml: float) -> float:
    """
    Total phenolic content (TPC) in mg GAE / g sample.

    Steps:
    1. Use calibration curve: C (mg GAE/mL) = (Abs - intercept) / slope
    2. Multiply by dilution factor -> mg GAE/mL in test solution
    3. Divide by extract concentration (mg extract / mL) to get mg GAE / mg extract
    4. Convert to mg GAE / g sample using extraction yield info (handled externally if needed)

    Here we return mg GAE per g sample directly if extract_conc_mg_per_ml is based on
    “mg extract from 1 g sample”.

    For example:
    - You dissolved 10 mg extract (obtained from 1 g sample) in 1 mL solvent:
      extract_conc_mg_per_ml = 10 mg/mL, and that corresponds to 1 g sample.
    """
    if slope == 0:
        raise ValueError("Slope cannot be zero.")
    if extract_conc_mg_per_ml == 0:
        raise ValueError("Extract concentration cannot be zero.")

    # Step 1: mg GAE/mL from calibration curve
    c_mg_gae_per_ml = (absorbance - intercept) / slope

    # Step 2: correct for dilution
    c_mg_gae_per_ml *= dilution_factor

    # Step 3: mg GAE per mg extract
    mg_gae_per_mg_extract = c_mg_gae_per_ml / extract_conc_mg_per_ml

    # Step 4: convert mg GAE/mg extract to mg GAE/g sample
    # If your extract_conc_mg_per_ml is defined as mg extract from 1 g sample per mL,
    # then mg_gae_per_mg_extract * 1000 gives mg GAE / g sample.
    mg_gae_per_g_sample = mg_gae_per_mg_extract * 1000.0

    return mg_gae_per_g_sample


def tfc_mg_qe_per_g_sample(absorbance: float,
                           slope: float,
                           intercept: float,
                           dilution_factor: float,
                           extract_conc_mg_per_ml: float) -> float:
    """
    Total flavonoid content (TFC) in mg QE / g sample.
    Exactly analogous to TPC function, but using catechin/quercetin equivalents.

    Calibration curve: C (mg QE/mL) = (Abs - intercept) / slope
    """
    if slope == 0:
        raise ValueError("Slope cannot be zero.")
    if extract_conc_mg_per_ml == 0:
        raise ValueError("Extract concentration cannot be zero.")

    c_mg_qe_per_ml = (absorbance - intercept) / slope
    c_mg_qe_per_ml *= dilution_factor
    mg_qe_per_mg_extract = c_mg_qe_per_ml / extract_conc_mg_per_ml
    mg_qe_per_g_sample = mg_qe_per_mg_extract * 1000.0
    return mg_qe_per_g_sample


def menu():
    while True:
        print("\n==== PHYTO CALCULATOR ====")
        print("1. Extraction yield (%)")
        print("2. DPPH radical scavenging (%)")
        print("3. Total phenolic content (TPC, mg GAE/g sample)")
        print("4. Total flavonoid content (TFC, mg QE/g sample)")
        print("5. Exit")

        choice = input("Choose an option (1-5): ").strip()

        try:
            if choice == "1":
                m_extract = float(input("Mass of dried extract (g): "))
                m_sample = float(input("Mass of initial sample (g): "))
                result = extraction_yield(m_extract, m_sample)
                print(f"Extraction yield = {result:.2f} %")

            elif choice == "2":
                a_ctrl = float(input("A_control: "))
                a_samp = float(input("A_sample: "))
                result = dpph_scavenging(a_ctrl, a_samp)
                print(f"DPPH scavenging = {result:.2f} %")

            elif choice == "3":
                abs_val = float(input("Absorbance of sample: "))
                slope = float(input("Calibration curve slope (GAE): "))
                intercept = float(input("Calibration curve intercept (GAE): "))
                df = float(input("Dilution factor: "))
                conc = float(input("Extract concentration (mg extract/mL): "))
                result = tpc_mg_gae_per_g_sample(abs_val, slope, intercept, df, conc)
                print(f"TPC = {result:.2f} mg GAE/g sample")

            elif choice == "4":
                abs_val = float(input("Absorbance of sample: "))
                slope = float(input("Calibration curve slope (QE): "))
                intercept = float(input("Calibration curve intercept (QE): "))
                df = float(input("Dilution factor: "))
                conc = float(input("Extract concentration (mg extract/mL): "))
                result = tfc_mg_qe_per_g_sample(abs_val, slope, intercept, df, conc)
                print(f"TFC = {result:.2f} mg QE/g sample")

            elif choice == "5":
                print("Exiting Phyto Calculator. Bye!")
                break

            else:
                print("Invalid choice, please try again.")

        except ValueError as e:
            print(f"Input error: {e}")


if __name__ == "__main__":
    menu()
