

```python
import pandas as pd
```


```python
schoolsdata = "Generators/PyCitySchools/generated_data/schools_complete.csv"
studentsdata = "Generators/PyCitySchools/generated_data/students_complete.csv"
```


```python
schooldata_df = pd.read_csv(schoolsdata)
```


```python
studentdata_df = pd.read_csv(studentsdata)
```


```python
#DISTRICT SUMMARY
#Total Schools
totalschools = schooldata_df["school_name"].count()
#Total Students
totalstudents = studentdata_df["student_name"].count()
#Total Budget
totalbudget = schooldata_df["budget"].sum()
#Average Math Score
avgmath = studentdata_df["math_score"].mean()
#Average Reading Score
avgreading = studentdata_df["reading_score"].mean()
#% Passing Math
passmath = studentdata_df[(studentdata_df["math_score"] > 70)].count()["math_score"]
failmath = studentdata_df[(studentdata_df["math_score"] <= 70)].count()["math_score"]
percpassmath = (passmath/(passmath + failmath))*100
#% Passing Reading
passreading = studentdata_df[(studentdata_df["reading_score"] > 70)].count()["reading_score"]
failreading = studentdata_df[(studentdata_df["reading_score"] <= 70)].count()["reading_score"]
percpassreading = (passreading/(passreading + failreading))*100
#Overall Passing Rate (Average of the above two)
overallpass = (percpassmath + percpassreading)/2
```


```python
#Order for District Summary
districtsummary = pd.DataFrame({"Total School": [totalschools],
                                   "Total Students": [totalstudents],
                                   "Total Budget": [totalbudget],
                                   "Average Math Score": [avgmath],
                                   "Average Reading Score": [avgreading],
                                   "% Passing Math":[percpassmath],
                                   "% Passing Reading":[percpassreading],
                                   "% Overall Passing Rate": [overallpass]})
```


```python
#Formatting for District Summary
districtsummary["Total Students"] = districtsummary["Total Students"].map("{0:,.0f}".format)
districtsummary["Total Budget"] = districtsummary["Total Budget"].map("${0:,.0f}".format)
districtsummary["% Passing Math"] = districtsummary["% Passing Math"].map("{0:,.2f}%".format)
districtsummary["% Passing Reading"] = districtsummary["% Passing Reading"].map("{0:,.2f}%".format)
districtsummary["% Overall Passing Rate"] = districtsummary["% Overall Passing Rate"].map("{0:,.2f}%".format)

ds = districtsummary[["Total School","Total Students","Total Budget","Average Math Score","Average Reading Score","% Passing Math","% Passing Reading", "% Overall Passing Rate"]]
ds
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total School</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11</td>
      <td>29,376</td>
      <td>$18,648,468</td>
      <td>82.269846</td>
      <td>82.865877</td>
      <td>84.12%</td>
      <td>76.81%</td>
      <td>80.47%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#dataframe with passing math score grouped by school name
math_df = studentdata_df.loc[studentdata_df["math_score"] > 70]
mathgroup_df = math_df.groupby(["school_name"]).count()
mathgroup_df.pop("Student ID")
mathgroup_df.pop("student_name")
mathgroup_df.pop("gender")
mathgroup_df.pop("grade")
mathgroup_df.pop("reading_score")
mathgroup_df.reset_index(inplace=True)
```


```python
#dataframe with passing reading score grouped by school name
read_df = studentdata_df.loc[studentdata_df["reading_score"] > 70]
readgroup_df = read_df.groupby(["school_name"]).count()
readgroup_df.pop("Student ID")
readgroup_df.pop("student_name")
readgroup_df.pop("gender")
readgroup_df.pop("grade")
readgroup_df.pop("math_score")
readgroup_df.reset_index(inplace=True)
```


```python
#Merge reading and math scores df
mathread = pd.merge(mathgroup_df, readgroup_df, left_on="school_name", right_on="school_name")
```


```python
#total students df
totstu = studentdata_df.loc[studentdata_df["Student ID"]]
totalstudentdf = totstu.groupby(["school_name"]).count()
totalstudentdf.pop("reading_score")
totalstudentdf.pop("student_name")
totalstudentdf.pop("gender")
totalstudentdf.pop("grade")
totalstudentdf.pop("math_score")
totalstudentdf.reset_index(inplace=True)
```


```python
#Calculate %math,reading,overall
tocalcperc = pd.merge(mathread, totalstudentdf, left_on="school_name", right_on="school_name")

percmath = (tocalcperc["math_score"] / tocalcperc["Student ID"])*100
tocalcperc["% Passing Math"] = percmath
percread = (tocalcperc["reading_score"] / tocalcperc["Student ID"])*100
tocalcperc["% Passing Reading"] = percread
percoverall = (percmath+percread)/2
tocalcperc["% Passing Overall"] = percoverall

perscentpassing = tocalcperc
```


```python
#only have %math,reading,overall
perscentpassing.pop("math_score")
perscentpassing.pop("reading_score")
perscentpassing.pop("Student ID");
```


```python
#avg reading score and avg math score
student2 = studentdata_df.groupby(["school_name"])
studentmean= student2.mean()
studentmean.pop("Student ID")
studentmean = studentmean.rename(columns={"reading_score":"Average Reading Score", "math_score":"Average Math Score"})
studentmean.reset_index(inplace=True)
```


```python
#total students, total school budget, per student budget
school2 = schooldata_df.groupby(["school_name","type"])
schoolsum = school2.mean()
schoolsum.pop("School ID")
schoolsum = schoolsum.rename(columns={"size":"Total Students", "budget":"Total School Budget"})
perstubud = schoolsum["Total School Budget"] / schoolsum["Total Students"]
schoolsum["Per Student Budget"] = perstubud
schoolsum.reset_index(inplace=True)
```


```python
#Merge perscentpassing, studentmean, schoolsum
perscentstudent = pd.merge(perscentpassing, studentmean, left_on="school_name", right_on="school_name")
perscentstudentschool = pd.merge(perscentstudent, schoolsum, left_on="school_name", right_on="school_name")
perscentstudentschool.set_index('school_name', inplace=True)
perscentstudentschool.index.name = None
perscentstudentschool
forscoresby = perscentstudentschool
forscoresby1 = perscentstudentschool.copy(deep=True)
forscoresby2 = perscentstudentschool.copy(deep=True)
```


```python
#Format & Order for School Summary
perscentstudentschool = perscentstudentschool.rename(columns={"type":"School Type"})

formatss = perscentstudentschool
formatss["Total School Budget"] = formatss["Total School Budget"].map("${:,.2f}".format)
formatss["Per Student Budget"] = formatss["Per Student Budget"].map("${:,.2f}".format)


schoolsummary = formatss[["School Type","Total Students","Total School Budget", "Per Student Budget", 
                      "Average Math Score","Average Reading Score","% Passing Math","% Passing Reading", "% Passing Overall"]]
schoolsummary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Campbell High School</th>
      <td>Charter</td>
      <td>271</td>
      <td>$157,993.00</td>
      <td>$583.00</td>
      <td>83.594096</td>
      <td>93.771218</td>
      <td>92.619926</td>
      <td>100.000000</td>
      <td>96.309963</td>
    </tr>
    <tr>
      <th>Galloway High School</th>
      <td>Charter</td>
      <td>2471</td>
      <td>$1,445,535.00</td>
      <td>$585.00</td>
      <td>83.566168</td>
      <td>94.029543</td>
      <td>90.813436</td>
      <td>100.000000</td>
      <td>95.406718</td>
    </tr>
    <tr>
      <th>Glass High School</th>
      <td>District</td>
      <td>3271</td>
      <td>$2,155,589.00</td>
      <td>$659.00</td>
      <td>81.293183</td>
      <td>76.888108</td>
      <td>79.333537</td>
      <td>65.117701</td>
      <td>72.225619</td>
    </tr>
    <tr>
      <th>Gomez High School</th>
      <td>Charter</td>
      <td>2154</td>
      <td>$1,288,092.00</td>
      <td>$598.00</td>
      <td>83.838440</td>
      <td>94.027391</td>
      <td>90.807799</td>
      <td>100.000000</td>
      <td>95.403900</td>
    </tr>
    <tr>
      <th>Gonzalez High School</th>
      <td>Charter</td>
      <td>1855</td>
      <td>$1,192,765.00</td>
      <td>$643.00</td>
      <td>83.442588</td>
      <td>94.140701</td>
      <td>89.649596</td>
      <td>100.000000</td>
      <td>94.824798</td>
    </tr>
    <tr>
      <th>Hawkins High School</th>
      <td>District</td>
      <td>4555</td>
      <td>$2,851,430.00</td>
      <td>$626.00</td>
      <td>81.723820</td>
      <td>77.005928</td>
      <td>81.668496</td>
      <td>64.983535</td>
      <td>73.326015</td>
    </tr>
    <tr>
      <th>Kelly High School</th>
      <td>District</td>
      <td>3307</td>
      <td>$2,225,611.00</td>
      <td>$673.00</td>
      <td>81.678258</td>
      <td>76.829755</td>
      <td>80.586634</td>
      <td>64.166919</td>
      <td>72.376777</td>
    </tr>
    <tr>
      <th>Macdonald High School</th>
      <td>Charter</td>
      <td>901</td>
      <td>$550,511.00</td>
      <td>$611.00</td>
      <td>83.779134</td>
      <td>93.932297</td>
      <td>92.230855</td>
      <td>100.000000</td>
      <td>96.115427</td>
    </tr>
    <tr>
      <th>Miller High School</th>
      <td>Charter</td>
      <td>2424</td>
      <td>$1,418,040.00</td>
      <td>$585.00</td>
      <td>83.610149</td>
      <td>93.997525</td>
      <td>90.800330</td>
      <td>100.000000</td>
      <td>95.400165</td>
    </tr>
    <tr>
      <th>Sherman High School</th>
      <td>District</td>
      <td>3213</td>
      <td>$2,152,710.00</td>
      <td>$670.00</td>
      <td>81.502023</td>
      <td>77.290694</td>
      <td>80.547775</td>
      <td>65.141612</td>
      <td>72.844693</td>
    </tr>
    <tr>
      <th>Smith High School</th>
      <td>District</td>
      <td>4954</td>
      <td>$3,210,192.00</td>
      <td>$648.00</td>
      <td>81.539160</td>
      <td>77.146952</td>
      <td>80.682277</td>
      <td>64.251110</td>
      <td>72.466694</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Top Performing Schools (By Passing Rate)
topschools = schoolsummary.sort_values("% Passing Overall", ascending=False)
topschools.head(5)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Campbell High School</th>
      <td>Charter</td>
      <td>271</td>
      <td>$157,993.00</td>
      <td>$583.00</td>
      <td>83.594096</td>
      <td>93.771218</td>
      <td>92.619926</td>
      <td>100.0</td>
      <td>96.309963</td>
    </tr>
    <tr>
      <th>Macdonald High School</th>
      <td>Charter</td>
      <td>901</td>
      <td>$550,511.00</td>
      <td>$611.00</td>
      <td>83.779134</td>
      <td>93.932297</td>
      <td>92.230855</td>
      <td>100.0</td>
      <td>96.115427</td>
    </tr>
    <tr>
      <th>Galloway High School</th>
      <td>Charter</td>
      <td>2471</td>
      <td>$1,445,535.00</td>
      <td>$585.00</td>
      <td>83.566168</td>
      <td>94.029543</td>
      <td>90.813436</td>
      <td>100.0</td>
      <td>95.406718</td>
    </tr>
    <tr>
      <th>Gomez High School</th>
      <td>Charter</td>
      <td>2154</td>
      <td>$1,288,092.00</td>
      <td>$598.00</td>
      <td>83.838440</td>
      <td>94.027391</td>
      <td>90.807799</td>
      <td>100.0</td>
      <td>95.403900</td>
    </tr>
    <tr>
      <th>Miller High School</th>
      <td>Charter</td>
      <td>2424</td>
      <td>$1,418,040.00</td>
      <td>$585.00</td>
      <td>83.610149</td>
      <td>93.997525</td>
      <td>90.800330</td>
      <td>100.0</td>
      <td>95.400165</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Bottom Performing Schools (By Passing Rate)
bottomschools = schoolsummary.sort_values("% Passing Overall")
bottomschools.head(5)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Glass High School</th>
      <td>District</td>
      <td>3271</td>
      <td>$2,155,589.00</td>
      <td>$659.00</td>
      <td>81.293183</td>
      <td>76.888108</td>
      <td>79.333537</td>
      <td>65.117701</td>
      <td>72.225619</td>
    </tr>
    <tr>
      <th>Kelly High School</th>
      <td>District</td>
      <td>3307</td>
      <td>$2,225,611.00</td>
      <td>$673.00</td>
      <td>81.678258</td>
      <td>76.829755</td>
      <td>80.586634</td>
      <td>64.166919</td>
      <td>72.376777</td>
    </tr>
    <tr>
      <th>Smith High School</th>
      <td>District</td>
      <td>4954</td>
      <td>$3,210,192.00</td>
      <td>$648.00</td>
      <td>81.539160</td>
      <td>77.146952</td>
      <td>80.682277</td>
      <td>64.251110</td>
      <td>72.466694</td>
    </tr>
    <tr>
      <th>Sherman High School</th>
      <td>District</td>
      <td>3213</td>
      <td>$2,152,710.00</td>
      <td>$670.00</td>
      <td>81.502023</td>
      <td>77.290694</td>
      <td>80.547775</td>
      <td>65.141612</td>
      <td>72.844693</td>
    </tr>
    <tr>
      <th>Hawkins High School</th>
      <td>District</td>
      <td>4555</td>
      <td>$2,851,430.00</td>
      <td>$626.00</td>
      <td>81.723820</td>
      <td>77.005928</td>
      <td>81.668496</td>
      <td>64.983535</td>
      <td>73.326015</td>
    </tr>
  </tbody>
</table>
</div>




```python
#MATH SCORES BY GRADE
mathbygrade = studentdata_df.groupby(["school_name","grade"]).mean()
mathbygrade.pop("Student ID")
mathbygrade.pop("reading_score")

mathbygrade1 = mathbygrade.groupby(["school_name","grade"])["math_score"].mean().unstack()
mathbygrade1= mathbygrade1[["9th","10th","11th","12th"]]
mathbygrade1.index.name = None
mathbygrade1 = mathbygrade1.rename_axis("", axis =1)
mathbygrade1
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Campbell High School</th>
      <td>83.842857</td>
      <td>84.269663</td>
      <td>83.940000</td>
      <td>82.064516</td>
    </tr>
    <tr>
      <th>Galloway High School</th>
      <td>83.534384</td>
      <td>83.551630</td>
      <td>83.975425</td>
      <td>83.204724</td>
    </tr>
    <tr>
      <th>Glass High School</th>
      <td>81.867647</td>
      <td>81.044652</td>
      <td>81.390935</td>
      <td>80.823120</td>
    </tr>
    <tr>
      <th>Gomez High School</th>
      <td>83.676568</td>
      <td>83.966817</td>
      <td>83.874468</td>
      <td>83.828916</td>
    </tr>
    <tr>
      <th>Gonzalez High School</th>
      <td>83.548263</td>
      <td>83.952118</td>
      <td>83.201970</td>
      <td>82.840206</td>
    </tr>
    <tr>
      <th>Hawkins High School</th>
      <td>81.667758</td>
      <td>81.475371</td>
      <td>81.885770</td>
      <td>81.938296</td>
    </tr>
    <tr>
      <th>Kelly High School</th>
      <td>81.789659</td>
      <td>81.881168</td>
      <td>81.497283</td>
      <td>81.453920</td>
    </tr>
    <tr>
      <th>Macdonald High School</th>
      <td>84.255507</td>
      <td>83.813953</td>
      <td>83.482906</td>
      <td>83.516484</td>
    </tr>
    <tr>
      <th>Miller High School</th>
      <td>83.823713</td>
      <td>83.624661</td>
      <td>83.635838</td>
      <td>83.304183</td>
    </tr>
    <tr>
      <th>Sherman High School</th>
      <td>81.496614</td>
      <td>81.526882</td>
      <td>81.232117</td>
      <td>81.735955</td>
    </tr>
    <tr>
      <th>Smith High School</th>
      <td>81.909804</td>
      <td>80.997980</td>
      <td>81.832724</td>
      <td>81.548182</td>
    </tr>
  </tbody>
</table>
</div>




```python
#READING SCORES BY GRADE
readbygrade = studentdata_df.groupby(["school_name","grade"]).mean()
readbygrade.pop("Student ID")
readbygrade.pop("math_score")

readbygrade1 = readbygrade.groupby(["school_name","grade"])["reading_score"].mean().unstack()
readbygrade1= readbygrade1[["9th","10th","11th","12th"]]
readbygrade1.index.name = None
readbygrade1 = readbygrade1.rename_axis("", axis =1)
readbygrade1
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Campbell High School</th>
      <td>93.471429</td>
      <td>93.876404</td>
      <td>94.080000</td>
      <td>93.709677</td>
    </tr>
    <tr>
      <th>Galloway High School</th>
      <td>94.065903</td>
      <td>93.961957</td>
      <td>93.979206</td>
      <td>94.129921</td>
    </tr>
    <tr>
      <th>Glass High School</th>
      <td>76.444570</td>
      <td>77.319834</td>
      <td>77.128895</td>
      <td>76.618384</td>
    </tr>
    <tr>
      <th>Gomez High School</th>
      <td>94.186469</td>
      <td>93.972851</td>
      <td>93.808511</td>
      <td>94.130120</td>
    </tr>
    <tr>
      <th>Gonzalez High School</th>
      <td>94.042471</td>
      <td>94.103131</td>
      <td>94.416256</td>
      <td>94.036082</td>
    </tr>
    <tr>
      <th>Hawkins High School</th>
      <td>76.518003</td>
      <td>77.174355</td>
      <td>77.526621</td>
      <td>76.852106</td>
    </tr>
    <tr>
      <th>Kelly High School</th>
      <td>76.367803</td>
      <td>77.267875</td>
      <td>76.637228</td>
      <td>76.966988</td>
    </tr>
    <tr>
      <th>Macdonald High School</th>
      <td>94.048458</td>
      <td>94.135659</td>
      <td>93.799145</td>
      <td>93.670330</td>
    </tr>
    <tr>
      <th>Miller High School</th>
      <td>93.897036</td>
      <td>94.039295</td>
      <td>94.238921</td>
      <td>93.823194</td>
    </tr>
    <tr>
      <th>Sherman High School</th>
      <td>77.292325</td>
      <td>77.111828</td>
      <td>77.312409</td>
      <td>77.501404</td>
    </tr>
    <tr>
      <th>Smith High School</th>
      <td>76.861176</td>
      <td>76.805387</td>
      <td>77.338208</td>
      <td>77.749091</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Scores by School Spending
forscoresby["Per Student Budget"].astype(float)
s_spending = forscoresby
s_spending.pop("type")
s_spending.pop("Total Students")
s_spending.pop("Total School Budget")
schoolspending = s_spending
schoolspending
spending_bins = [0, 585, 615, 645, 675]
spending_group = ['<$585', '$585-615', '$615-645', '$645-675']
schoolspending["Spending Ranges (Per Student)"] = pd.cut(schoolspending["Per Student Budget"], spending_bins, labels=spending_group)
schoolspending_groups = schoolspending.groupby("Spending Ranges (Per Student)")
ssg = schoolspending_groups[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading", "% Passing Overall"]]
ssg.max()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>83.610149</td>
      <td>94.029543</td>
      <td>92.619926</td>
      <td>100.000000</td>
      <td>96.309963</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>83.838440</td>
      <td>94.027391</td>
      <td>92.230855</td>
      <td>100.000000</td>
      <td>96.115427</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>83.442588</td>
      <td>94.140701</td>
      <td>89.649596</td>
      <td>100.000000</td>
      <td>94.824798</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>81.678258</td>
      <td>77.290694</td>
      <td>80.682277</td>
      <td>65.141612</td>
      <td>72.844693</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Scores by School Size
s_size = forscoresby1
s_size.pop("type")
s_size.pop("Per Student Budget")
s_size.pop("Total School Budget")
schoolsize = s_size
schoolsize
size_bins = [0, 1000, 2000, 5000]
size_group = ['Small(<1000)', 'Medium (1000-2000)', 'Large (2000-5000)']
schoolsize["School Size"] = pd.cut(schoolsize["Total Students"], size_bins, labels=size_group)
schoolsize_groups = schoolsize.groupby("School Size")
ssizeg = schoolsize_groups[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading", "% Passing Overall"]]
ssizeg.max()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small(&lt;1000)</th>
      <td>83.779134</td>
      <td>93.932297</td>
      <td>92.619926</td>
      <td>100.0</td>
      <td>96.309963</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>83.442588</td>
      <td>94.140701</td>
      <td>89.649596</td>
      <td>100.0</td>
      <td>94.824798</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>83.838440</td>
      <td>94.029543</td>
      <td>90.813436</td>
      <td>100.0</td>
      <td>95.406718</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Scores by School Type
s_type = forscoresby2
s_type.pop("Total Students")
s_type.pop("Per Student Budget")
s_type.pop("Total School Budget")
schooltype = s_type
typegroup = schooltype.groupby(["type"]).mean()
typegroup = typegroup[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading", "% Passing Overall"]]
typegroup.index.names = ['School Type']
typegroup
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.638429</td>
      <td>93.983112</td>
      <td>91.153657</td>
      <td>100.000000</td>
      <td>95.576828</td>
    </tr>
    <tr>
      <th>District</th>
      <td>81.547289</td>
      <td>77.032287</td>
      <td>80.563744</td>
      <td>64.732175</td>
      <td>72.647960</td>
    </tr>
  </tbody>
</table>
</div>



Three Observable Trends:
1. Schools with smaller student body size have a higher percentage of passing students.
2. Charter school students are more likely to succeed in math and reading than district school students.
3. Schools that have higher spending ranges per student are more likely to have failing reading scores compared to students in schools with lower spending ranges.

