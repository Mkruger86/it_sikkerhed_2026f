# It-Sikkerhed 2026, Skole projekt på Zealand, Næstved

## 1. *Run tests med ctrl+shift+b uden nogen initielle ændringer*  
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/8f220abb-85b3-4fa2-a62f-ebd76d6f82d7" />

## 2. *Parametrized AB tests med low/high edge cases*
### Kode
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e721f3ca-8f9b-418e-a4d8-e260980d22c6" />
### Output
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/a3890ec6-e5d3-4ca8-915f-8272fbba548a" />
#### Forklaring
Med @pytest.mark.parametrize("a,b", AB_CASES) kører pytest den samme testfunktion flere gange, en gang pr. case, og tildeler værdierne til a og b i hvert run. 
Det gør det muligt at teste flere inputkombinationer uden at skrive flere testfunktioner; hver case bliver en separat testrun (med eget id og evt. xfail).

## 3. *Fixture AB tests, igen med edge cases
### Kode
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/fa21dec8-4953-4cbb-8018-a608a9b1351d" />
### (samme) Output
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/cb988bd5-08b6-40fa-bf75-63d628584338" />
#### Forklaring
Med @pytest.fixture(params=AB_CASES) bliver fixture ab kørt en gang pr. element i AB_CASES, og den aktuelle værdi kan hentes som request.param (fx (3, 2)). Hver test der tager ab som argument (def test_x(ab): ...) udføres derfor automatisk for alle cases, hvor a, b = ab giver input til beregningen.
