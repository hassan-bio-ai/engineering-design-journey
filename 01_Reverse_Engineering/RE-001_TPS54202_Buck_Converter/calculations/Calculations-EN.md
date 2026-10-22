# Power Supply Design Calculations

This file documents the mathematical formulas and engineering equations upon which the actual values of the electronic components in the (TIDA-00946) reference power board design based on the (TPS54202) IC were selected. The fundamental parameters for our project are:
*   **Nominal Input Voltage ($V_{IN}$):** $12V$ (with operating capability within the $6.5V$ to $20V$ range)
*   **Target Output Voltage ($V_{OUT}$):** $5V$
*   **Maximum Output Current ($I_{OUT}$):** $1A$
*   **Switching Frequency ($f_{sw}$):** $500kHz$ (internally fixed value in the chip)
*   **Internal Reference Voltage ($V_{REF}$):** $0.596V$

---

## 1. Feedback Voltage Divider Calculation

**Objective:** Determine the values of the upper ($R_3$) and lower ($R_6$) resistors to ensure the output voltage is perfectly stabilized at $5V$, matching the values selected by the reference board designers.

### The Formula
The IC relies on an error amplifier that compares the voltage coming from the voltage divider with the internal reference voltage. The output voltage equation is given as follows:

$$V_{OUT} = V_{REF} \times \left( 1 + \frac{R_3}{R_6} \right)$$

To find the value of the upper resistor ($R_3$) given the lower resistor, we rearrange the equation:

$$R_3 = R_6 \times \left( \frac{V_{OUT}}{V_{REF}} - 1 \right)$$

### Application & Designer's Choice
1. **Selecting the Lower Resistor Value ($R_6$):** 
   From an engineering perspective, any value can be chosen for the lower resistor. Small values (like $10k\Omega$) provide excellent stability against noise but draw a continuous current from the output to the ground, which reduces Light Load Efficiency. 
   To maximize the chip's efficiency and optimally activate the power-saving mode, the designer chose a higher standard value for the lower resistor: **$R_6 = 26.7 k\Omega$**.

2. **Calculating the Upper Resistor ($R_3$):**
   Substituting the designer's value into the derived equation to calculate the required upper resistor:
   $$R_3 = 26,700 \times \left( \frac{5}{0.596} - 1 \right)$$
   $$R_3 = 26,700 \times (8.3892 - 1)$$
   $$R_3 = 26,700 \times 7.3892 \approx 197,291 \Omega \approx 197.3 k\Omega$$

3. **Selecting the Standard Resistor Value (Standard 1% Resistor):**
   The closest standard value available in the market (E96 series with 1% tolerance) for $197.3k\Omega$ is **$200 k\Omega$**.
   
   To verify the accuracy of the actual output when using these resistors ($R_3 = 200 k\Omega$ and $R_6 = 26.7 k\Omega$):
   $$V_{OUT(Actual)} = 0.596 \times \left( 1 + \frac{200}{26.7} \right)$$
   $$V_{OUT(Actual)} = 0.596 \times (1 + 7.4906) = 0.596 \times 8.4906 = 5.06 V$$
   The actual voltage ($5.06V$) is considered highly accurate and within the acceptable tolerance for powering sensitive digital loads.

**Final Result for Section 1:**
*   **Upper Resistor $R_{FB\_Upper} (R_3) = 200 k\Omega$**
*   **Lower Resistor $R_{FB\_Lower} (R_6) = 26.7 k\Omega$**


## 2. Enable Divider and Under-Voltage Lockout (UVLO) Calculation

**Objective:** Analyze the values chosen by the designer in the reference board for the upper ($R_2 = 562 k\Omega$) and lower ($R_5 = 102 k\Omega$) resistors connected to the `EN` enable pin, and derive the "Safe Turn-on Voltage" ($V_{START}$) at which the system will permit switching to begin.

### The Formula
The `EN` enable pin relies on an internal voltage comparator with a precise reference turn-on threshold of approximately ($V_{EN\_Threshold} \approx 1.21V$) in this family of ICs. 
To simplify the calculation (ignoring the minor internal microampere hysteresis currents), the overall system startup voltage ($V_{START}$) equation is given as:

$$V_{START} = V_{EN\_Threshold} \times \left( 1 + \frac{R_2}{R_5} \right)$$

### Application & Calculation
Substituting the actual values found in the practical schematic:
$$V_{START} = 1.21 \times \left( 1 + \frac{562}{102} \right)$$
$$V_{START} = 1.21 \times (1 + 5.5098)$$
$$V_{START} = 1.21 \times 6.5098 \approx 7.87 V$$

### Engineering Notes on Designer's Choice
The calculations reveal that the circuit will not start until the voltage reaches approximately ($7.87V$), even though the (TPS54202) chip is theoretically capable of operating with a voltage starting from $4.5V$ or $6.5V$. This restriction is intentional and is based on two engineering justifications:

1. **7805 Drop-in Replacement Simulation:** As shown in the PCB layout analysis, this module is designed to be a direct drop-in replacement for the old (7805) linear regulator. Physically, the 7805 requires a Dropout Voltage of about $2V$ to $2.5V$ to function properly (i.e., it does not output a stable $5V$ unless the input is $7.5V$ or higher). For the modern switching board to behave identically as a compatible replacement, the designer deliberately configured it to shut down if the voltage drops below $7.8V$.
2. **12V Systems Protection:** These boards are frequently used with batteries or $12V$ supply rails. A voltage drop to levels around $6.5V$ in most systems means the battery is nearly empty (damaged) or there is a short circuit somewhere causing this dangerous drop. Therefore, setting the startup threshold at $7.87V$ acts as an excellent Under-Voltage Lockout (UVLO) protection measure, preventing the system from operating and drawing high current under unstable power conditions, thereby preventing circuit chattering.

**Final Result for Section 2:**
*   **Safe Startup Threshold ($V_{START}$): $\approx 7.87 V$**
*   **Achieved via the resistor network: $R_2 = 562 k\Omega$ and $R_5 = 102 k\Omega$**


## 3. Output Smoothing Inductor (L1) Calculation

**Objective:** Derive the mathematical value of the output inductor ($L_1$) and prove the engineering reasoning that led the designer to select the standard value of ($22\mu H$) present in the practical schematic.

### The Formula
The inductor value in Buck Converters is calculated to ensure the "Inductor Ripple Current" ($\Delta I_L$) does not exceed specific limits. The standard engineering equation for calculating the inductor value is:

$$L = \frac{V_{OUT} \times (V_{IN} - V_{OUT})}{V_{IN} \times \Delta I_L \times f_{sw}}$$

Where:
*   $V_{IN}$: Input voltage (We will use the nominal voltage of $12V$).
*   $V_{OUT}$: Target output voltage ($5V$).
*   $f_{sw}$: IC switching frequency ($500kHz$ or $500,000 Hz$).
*   $\Delta I_L$: Allowable ripple current inside the inductor.

**How to determine $\Delta I_L$?**
Engineering-wise, the golden rule in power supply design is to select a ripple current ranging between $20\%$ to $40\%$ of the maximum output current ($I_{OUT}$). 
Since our circuit's maximum output current is $1A$, we will choose an ideal and balanced ratio of $30\%$ (i.e., coefficient $K_{IND} = 0.3$).
Thus:
$$\Delta I_L = 1A \times 0.3 = 0.3 A$$

### Application & Calculation
Substituting all values into the primary equation:

$$L = \frac{5 \times (12 - 5)}{12 \times 0.3 \times 500,000}$$

$$L = \frac{5 \times 7}{12 \times 150,000}$$

$$L = \frac{35}{1,800,000}$$

$$L \approx 0.0000194 H \approx 19.4 \mu H$$

### Engineering Notes on Designer's Choice
The precise calculation indicates that we need an inductor with a value of $19.4 \mu H$. However, in the electronics world, components are manufactured in fixed Standard Values. 

1. **Rounding to the Next Standard Value:** The closest commercially available standard values are $15\mu H$ and $22\mu H$. The designer selected the higher value, which is **$22\mu H$**.
2. **Engineering Justification:** Rounding up to the higher value (from $19.4$ to $22$) is a very smart engineering decision; physically increasing the inductor's value reduces the "ripple current" ($\Delta I_L$) even further than the $30\%$ ratio we assumed. Reducing ripples means obtaining a much more stable and pure direct current (DC) at the output, which is crucial for powering sensitive loads in embedded systems.

**Final Result for Section 3:**
*   **Theoretically Calculated Value:** $19.4 \mu H$
*   **Actual Value Selected for Design:** $L_1 = 22 \mu H$


## 4. Output Capacitors (C4 & C5) Calculation

**Objective:** Derive the minimum required output capacitance ($C_{OUT}$) to ensure voltage stability, and explain the engineering rationale behind the designer's choice to use two parallel capacitors of ($22\mu F$) each (totaling $44\mu F$).

### The Formula
The output capacitor's capacitance is calculated based on the allowable "Output Voltage Ripple" ($\Delta V_{OUT}$). When using ceramic capacitors (where ESR is almost zero), the equation for the minimum required capacitance is given as follows:

$$C_{OUT(min)} = \frac{\Delta I_L}{8 \times f_{sw} \times \Delta V_{OUT}}$$

Where:
*   $\Delta I_L$: Inductor ripple current calculated previously ($0.3 A$).
*   $f_{sw}$: Switching frequency ($500,000 Hz$).
*   $\Delta V_{OUT}$: Maximum allowable voltage ripple. Engineering-wise, it is preferred that the ripple does not exceed $0.5\%$ to $1\%$ of the output voltage ($5V$) for sensitive loads. We will assume a very strict ripple of $25mV$ (or $0.025V$).

### Application & Calculation
Substituting the values into the equation:

$$C_{OUT(min)} = \frac{0.3}{8 \times 500,000 \times 0.025}$$

$$C_{OUT(min)} = \frac{0.3}{100,000} = 0.000003 F = 3 \mu F$$

### Engineering Notes on Designer's Choice
Pure theoretical calculations showed that we only need a capacitor with a capacitance of $3 \mu F$ to smooth out the ripples! However, looking at the practical schematic, we find that the designer used two capacitors ($C_4$ and $C_5$), each valued at $22\mu F$ (a total of $44\mu F$). 
This massive difference between the theoretical calculation ($3\mu F$) and the practical application ($44\mu F$) is due to three critical engineering reasons:

1. **DC Bias Derating Effect:** As mentioned in the architecture analysis, Multi-Layer Ceramic Capacitors (MLCC) lose a significant portion of their actual capacitance when a DC voltage is applied across them. A capacitor with a nominal value of $22\mu F$ might have its actual capacitance drop to $10\mu F$ or less when $5V$ is continuously applied to it. Therefore, the designer doubled the capacitance to compensate for this inevitable loss.
2. **Transient Response:** The previous equation calculates the "Steady-State" condition. But what if the system suddenly draws a high current? A sudden Voltage Dip will occur. The large capacitance ($44\mu F$) acts as a massive energy reservoir that intervenes in microseconds to compensate for this deficit before the internal error amplifier can respond, thereby maintaining system stability.
3. **Reducing Total ESR:** Placing two capacitors in parallel halves the Equivalent Series Resistance (ESR), which reduces dissipated heat and makes the high-frequency filtering process vastly more efficient than using a single large capacitor.

**Final Result for Section 4:**
*   **Theoretically Calculated Minimum Capacitance:** $3 \mu F$
*   **Actual Capacitance Selected for Design:** $C_4 + C_5 = 44 \mu F$ (To overcome DC Bias and improve Transient Response).


## 5. Engineering Verdict

The calculations documented in this file prove that the selection of values and electronic components for the power board was not arbitrary, but rather based on precise engineering standards that balance pure theoretical performance with real-world operating environment requirements. 

Through this series of calculations, the safety and reliability of the design have been confirmed across several key aspects:
* **Absolute Regulation Accuracy:** Ensuring the output voltage is completely stable at $5V$ to power digital loads, thanks to the precise tuning of the feedback network.
* **Reliability and Proactive Protection:** Pre-setting the system startup threshold (UVLO) at $7.87V$ to prevent circuit chattering and protect the power source from excessive current draw under low-power conditions.
* **Power Quality:** Selecting a $22\mu H$ smoothing inductor to minimize ripple current, and doubling the output capacitance to $44\mu F$ as a preventive engineering measure to absorb violent momentary changes in current draw (Transient Response) and completely overcome the DC Bias Derating phenomenon.

This rigorous mathematical methodology ensures the ability of the step-down converter unit (TPS54202) to provide continuous, pure, and highly efficient power, making it the solid engineering foundation for operating complex sensors in this system without any concerns about performance fluctuation or thermal collapse.