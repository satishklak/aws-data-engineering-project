# aws-data-engineering-project

Goal of every DataEngineering project is to generate business value , so reason to collect diff types of data and get insights
Our client passionate about music industry, want to understand by collecting different types of data so that he can find some patterns and make music based on that
spotify we have multiple playlist available and client wants to start with top global songs types of songs, albums treding, who is the artists who is making all of these songs
this playlist updates every weekly, he wants to build dataset thru out the year, so build etl pipeline , weekly basis collect data after 2 yrs to get insights

steps
1.spotify api (spotify app has artist, album and track/songs info) - provided by spotify, register to get client and secret id to collect data , use pckg spotify
2.write code and deploy into aws lambda (compute service run on different event basis)
3.on top run cloudwatch trigger (run code on day or weekly basis ataspecific time)( runs lambda code extracts data and puts into s3)
4.s3 obj storage raw data store, then write transf code ; auto triggers trans function when data put on s3 and put data into s3 again
5.once data on s3 glue crawler goes thru each of file understands hwo many cols it has, col names, data types of each col, understand file as it is and will build a Glue catalog (has meta infor about data)
6.use Athena to directly run  sql query on top of it

tasks 
extract data from spotifyapi on weekly basis need trigger on weekly basis , need compute service lambda extract and put some data on s3 location,
then transorm and put some data on s3 location, glue crawler to build data catalog to run sql queries on top of it;
any ETL tool can be put to do same operation instead of lambda; chosen completely serverless architecture

Client ID : **************
Client secret : ****************

we have crednetials, write python code now
use different funcs in https://spotipy.readthedocs.io/en/2.22.1/ to extract code
album href id, name, releasedate ,totaltracks, externalurls spotify

complete this package and properly convert this into proper dataframe ; do for all of the albums in the list using loop
loop thru all the items and get required details

now deploy this into cloud and automate to run sql queries on top of it

now register in aws cloud , set billing preferences alerts enabled, go to cloudwatch create alrm billing metric > rs5 create topic with email
create spotify-etl-project-satishkl1 s3 bucket, create folder raw_data (stores json as is) create /to_processed, /processed folders, transformed_data
create folder /album_data, /songs_data, /artist_data ; object storage that can be stored unlimited files - json,csv etc.,
now deploy code to lambda - run small code, trans job reate event based trigger
create function spotify_api_data_extract , testevent1, configuration permissions add env vars ; 
to use external functions we have to use lambda layer rather pip install locally
create lambda layer spotipy_layer upload spotipy_layer.zip, click x86_64, python 3.8
add custom layer in lambda code function above


