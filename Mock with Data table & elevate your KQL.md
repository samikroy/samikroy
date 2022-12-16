let NewUsers = datatable (Users :string,  LocationDetails: dynamic )[
"developer.user@nodomain.com", dynamic({"city":"Bengaluru","state":"Karnataka","countryOrRegion":"IN"}),
"support.user@nodomain.com", dynamic({"city":"Hyderabad","state":"Telengana","countryOrRegion":"IN"})
];
union
NewUsers,
SigninLogs
| extend City= tostring(LocationDetails.city)
| extend State= tostring(LocationDetails.state)
| extend Country= tostring(LocationDetails.countryOrRegion)
| summarize count() by City, State, Country
