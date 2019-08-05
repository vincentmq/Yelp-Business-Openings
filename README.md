# Yelp-Business-Openings
## Background
I was asked to merge 12 files presumably webscraped from Yelp that detail unique business information. The 12 files represent 12 "rounds". These rounds are presumed to be survey rounds.

**The Task**: A number of key variables were to be kept and harmonized and then merged together on a unique business_id variable.

## Setup
Originally sent as CSV files, I was told that the files were actually JSON that were converted to CSV. Probably due to some problems with JSON -> CSV conversion, importing the CSVs into Stata resulted in a host of blank and unrecognizable datapoints. Thus I manually converted all CSV to XLSX. This fixed all importing problems.

## Execution
The starting files are

	yelp\_business\_round`i'.dta
		
1. Harmonizing Variables

		cd "$maindir/original_xlsx"
			forval i = 2/13{
				import excel using "yelp_business_round`i'.xlsx", firstrow clear
				capture rename is_open open`i'
				capture rename open open`i'
				capture rename address full_address
				drop if missing(business_id)
				keep business_id name full_address city longitude latitude open`i'
				save "$maindir/clean_dta/yelp_business_round`i'.dta", replace
			}
			
2. Merging 

		cd "$maindir/clean_dta"
			use yelp_business_round2.dta, clear
			forval i=3/13{
				merge 1:1 business_id using yelp_business_round`i'.dta
				keep if _merge==3
				drop _merge
			}
After merging, I noticed that in the end, I was getting no observations.

3. Investigate why merging resulted in no observations
 
		use yelp_business_round2.dta, clear
			forval i=3/13{
				merge 1:1 business_id using yelp_business_round`i'.dta
				rename _merge _merge`i'
			}

	|round	|count|merge==3 for all|
	|:---|:----|:-----|
	|2|	11375|	 *start*   |
	|3|	15393|	9544|
	|4|	41633|	9344|
	|5|	60431|	9223|
	|6|	60431|	9223|
	|7|	76464|	9071|
	|8|	84779|	8940|
	|9|142249|	0|
	|10|	154632|	0|
	|11|	172332|	0|
	|12|	186206|	0|
	|13|	190184|	0|

4. It looks like the problem starts when 9 is merged onto the rest of the data. So I decided to merge 2-8 and 9-13, creating two files with a large number of successful merges.

		use yelp_business_round2.dta, clear
			merge 1:1 business_id using yelp_business_round3.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round4.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round5.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round6.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round7.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round8.dta
				keep if _merge==3
				drop _merge
		save "$maindir/merge_dta/yelp_business_round_2_8.dta", replace
		
		use yelp_business_round9.dta, clear
			merge 1:1 business_id using yelp_business_round10.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round11.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round12.dta
				keep if _merge==3
				drop _merge
			merge 1:1 business_id using yelp_business_round13.dta
				keep if _merge==3
				drop _merge
		save "$maindir/merge_dta/yelp_business_round9_13.dta",replace

	|round	|merge==3 when merging 2 to 8 / 9 to 13|
	|:---|:---|
	|2|	*start*|
	|3|	9544|
	|4|	9344|
	|5|	9223|
	|6|	9223|
	|7|	9071|
	|8|	8940|
	|9|	*start*|
	|10|	140175|
	|11|	139320|
	|12|	131565|
	|13|	128752|

## Conclusion
As of today [8/3/2019] I sent the data off to Jiyeon and who will ask Hao what to do next. 
