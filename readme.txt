                                R E A D M E

#### files used:

pombola.json




#### files created:

personId.csv
orgId.csv
charges.csv
spreadsheet.xlsx




##### extract data from pombola.json

bash:

jq '.persons[] | [.id, .sort_name, .name, .images, .other_names]' -c pombola.json > personId.csv

## this extracts person-based info for later matching in 'charges'
## note: 'other_names' hasn't been used yet but should be cleaned up and consolidated;
##       some of the 'other_names' entries contain new info


jq '.organizations[] | [.id, .name, .category, .classification]' -c pombola.json > orgId.csv

## this extracts org-based information for later matching in 'charges'
## note: 'category' and 'classification' haven't been used yet but could potentially be useful


jq '.persons[] | .memberships[] | .id, .person_id, .role, .organization_id, {startDate: .start_date}, {endDate: end_date}, .area.area_type, .area.id, .area.identifier, .area.name]' -c pombola.json > charges.csv

## this extracts charges ie roles, which are the rows in the spreadsheet
## note1: .id isn't used but may be useful (unique per role)
## note2: 'person_id' and 'organization_id' are used to lookup person-based and org-based details, 
##        and will be used in the more better version of this process too
## note3: - startDate and endDate are extracted as tuples to avoid Excel reformatting them;
##          this will have to be made less clumsy eventually
##		  - both startDate and endDate have one of the following lengths: 
##		    0 (null), 4 (year only), 7 (year and month), 10 (full date)
##  	    (i double checked by extracting each group and adding up their rows)
##          so it's definitely possible to do the following and combine, but I'm not convinced it's the best approach:
			## see 'note3.txt' ##
## note4: 'area.area_type', 'area.id', and 'area.identifier' aren't used but may come in handy;
          only 'area.name' is currently used for the Territory column




#### paste data into spreadsheet.xlsx
- the 3 csv files created above are all pasted into separate sheets in the spreadsheet.xlsx workbook (by hand!)
- remove [s, ]s and "s so that the orgId and personId values match each other in the different sheets
  (used find/replace all per column)
- orgId and personId sheets are sorted to enable VLOOKUPs
- in a new 'allInfo' sheet, cell references and VLOOKUPs are used to match the person-based and org-based info per charge:

  =VLOOKUP(charges!B2, personInfo!$A$2:$B$6989, 2) ## gives surname
  =VLOOKUP(charges!B2, personInfo!$A$2:$C$6989, 3) ## gives full name
  =charges!C2                                      ## gives role
  =VLOOKUP(charges!D2, orgInfo!$A$2:$B$1320, 2)    ## gives org name
  =charges!N2                                      ## gives territory name
  =charges!G2                                      ## gives start date in form {"startDate":null} or {"startDate":"2014-05-21"} etc
  =charges!J2                                      ## gives end date in form {"endDate":null} or {"endDate":"2014-05-21"} etc
  =VLOOKUP(charges!B2,personInfo!$A$2:$D$6989, 4)  ## gives photo url
  
- in 'allInfoValues', the results from allInfo are pasted sans equations for hopefully easy pasting;
  ]s are removed from the Territory column,
  {"startDate": , {"endDate": and } are removed from the two Date columns, and
  [{"url":" and "}] are removed from the PhotoUrl column
  (used find/replace all per column);
  also included the (blank) columns that we don't have values for so that our values will end up in the correct columns


  
that's it so far! there's definitely better ways of doing this but this is the knowledge i had at my disposal :)
