# CODE
* Start by turning on the log  
log using "MEL_Manager_Interview_Analysis.log", replace  

* Question 1  
* a. Import dataset1 into Stata with the first row as variable names  
import delimited "C:\Users\Water\Desktop\MEL Manager Written Test\MEL Manager\dataset1.csv"

* b. How many observations and variables are in the dataset?  
describe // This will show the number of variables and observations  

* c. What is the unit of observation?  
* The unit of observation is Household.  

* Question 2  
* a. Data contains some duplicates in terms of household ID (hh_id). How many duplicates are there in this dataset?  
duplicates report hh_id  

* b. List country hh_id and respondent given name 1 for all duplicate observations  
duplicates report hh_id hh_name
duplicates tag hh_id hh_name, generate (duplicate_tag)
list hh_id hh_name if duplicate_tag > 0 

* c. Drop excess observations  
* Drop the observation where hh_id is 174179 and respondent is Scovia  
drop if hh_id == 174179 & respondent_given_name_1 == "Scovia"  

* Drop the observation where hh_id is 178903 and respondent is David  
drop if hh_id == 178903 & respondent_given_name_1 == "David"  

* Drop the observation where hh_id is 187589 and respondent is Immaculate  
drop if hh_id == 187589 & respondent_given_name_1 == "Immaculate"  

* d. What is the proportion of households from Uganda?  
egen total_uganda = total(country == "Uganda")  
egen total_households = total(!missing(hh_id))  
display total_uganda / total_households  

* Question 3  
* a. Rename the following variables  
rename how_much_does_a_unit_of_sweet_po ucost_sweetpotatoes  
rename quantity_other_food quantity_otherfood  
rename ucode_other_food ucode_otherfood  
rename consumption_avocado consumption_avocados  
rename consumption_otherfruitveg_descri specify_otherfruitveg_descri  

* Save the clean dataset as a dta file with the name “Dataset1 clean”  
save "Dataset1_clean.dta", replace  

* b. Reshape the data to a long format for all consumption_ quantity_ ucode_ ucost_  
reshape long consumption_ quantity_ ucode_ ucost_, i(hh_id) j(Product)  

* c. How many observations and variables now exist?  
describe  

* Question 4  
* a. Replace both quantity and ucost to zero, where there was no consumption (consumption=No)  
replace quantity = 0 if consumption == "No"  
replace ucost = 0 if consumption == "No"  

* b. Replace both quantity and ucost to missing where the quantity captured is either equal to 99, 98 or 999  
replace quantity = . if quantity == 99 | quantity == 98 | quantity == 999  
replace ucost = . if ucost == 99 | ucost == 98 | ucost == 999  

* c. For all values of ucost less than 100 shillings in Uganda, replace them to missing  
replace ucost = . if ucost < 100 & country == "Uganda"  

* d. For all values of ucost less than 5 shillings in Kenya, replace them to missing  
replace ucost = . if ucost < 5 & country == "Kenya"  

* Question 5  
* a. By product and country, calculate the median ucost and quantity  
bysort Product country: egen median_ucost = median(ucost)  
bysort Product country: egen median_quantity = median(quantity)  

* b. Replace all missing values with median ucost and median quantity  
replace ucost = median_ucost if missing(ucost)  
replace quantity = median_quantity if missing(quantity)  

* c. Calculate the weekly expenditure on each item and store it in a variable “Weekly_exp”  
gen Weekly_exp = quantity * ucost  

* d. Calculate the total expenditure per household.   
collapse (sum) total_expenditure = Weekly_exp, by(hh_id)  

* Question 6  
* a. Merge the resultant data with “Dataset1 clean”  
merge 1:1 hh_id using "Dataset1_clean.dta"  

* b. How many households currently save money?  
count if currently_save == 1  

* c. Convert the saved amount into dollars (\$1 = Ugx 3650, \$1 = Ksh 110)  
gen saved_amount_usd = (saved_amount / 3650) if country == "Uganda"  
replace saved_amount_usd = (saved_amount / 110) if country == "Kenya"  

* d. Calculate the average savings per country  
bysort country: egen avg_savings = mean(saved_amount_usd)  

* e. Test the hypothesis that the savings for females are equal to the savings for males.  
ttest saved_amount_usd, by(gender)  

* End of the analysis  
log close  
