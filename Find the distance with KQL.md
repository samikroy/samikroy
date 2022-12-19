let BaseLocationlatitude = 0.0;
let BaseLocationlongitude = 0.0;
SigninLogs
| extend latitude_ = tolong(LocationDetails.geoCoordinates.latitude)
| extend longitude_ = tolong(LocationDetails.geoCoordinates.longitude)
| extend Location = strcat(LocationDetails.city,', ' ,LocationDetails.state,', ', LocationDetails.countryOrRegion)
| distinct latitude_, longitude_, Location
| take 10
| extend distance_in_meters = geo_distance_2points(longitude_, latitude_,BaseLocationlongitude,BaseLocationlatitude)
| extend FinalString =strcat('🗺️ ', Location , ' is ', distance_in_meters/1000.0, ' KM away from the Center of the 🌎.')
| project FinalString
