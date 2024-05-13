# AWS-DATA Engineering Project

Goal of every DataEngineering project is to generate business value , so reason to collect diff types of data and get insights
Our client passionate about music industry, want to understand by collecting different types of data so that he can find some patterns and make music based on that
spotify we have multiple playlist available and client wants to start with top global songs types of songs, albums treding, who is the artists who is making all of these songs
this playlist updates every weekly, he wants to build dataset thru out the year, so build etl pipeline , weekly basis collect data after 2 yrs to get insights

### Architecture
![Architecture Diagram](https://github.com/satishklak/aws-data-engineering-project/blob/main/Architecture.jpg)

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

we have crednetials, write python code now, use different funcs in https://spotipy.readthedocs.io/en/2.22.1/ to extract code - album href id, name, releasedate ,totaltracks, externalurls spotify
complete this package and properly convert this into proper dataframe ; do for all of the albums in the list using loop , loop thru all the items and get required details
now deploy this into cloud and automate to run sql queries on top of it
now register in aws cloud , set billing preferences alerts enabled, go to cloudwatch create alrm billing metric > rs5 create topic with email
create spotify-etl-project-satishkl1 s3 bucket, create folder raw_data (stores json as is) create /to_processed, /processed folders, transformed_data
create folder /album_data, /songs_data, /artist_data ; object storage that can be stored unlimited files - json,csv etc.,
now deploy code to lambda - run small code, trans job reate event based trigger
create function spotify_api_data_extract , testevent1, configuration permissions add env vars ; 
to use external functions we have to use lambda layer rather pip install locally
create lambda layer spotipy_layer upload spotipy_layer.zip, click x86_64, python 3.8 , add custom layer in lambda code function above

import json
import os
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials

def lambda_handler(event, context):

    cilent_id = os.environ.get('client_id')
    client_secret = os.environ.get('client_secret')
    
    client_credentials_manager = SpotifyClientCredentials(client_id=cilent_id, client_secret=client_secret)
    sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)
    playlists = sp.user_playlists('spotify')
    
    playlist_link = "https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF?si=1333723a6eff4b7f"
    playlist_URI = playlist_link.split("/")[-1].split("?")[0]
    
    spotify_data = sp.playlist_tracks(playlist_URI)
    
    print(spotify_data)

    same json data able to print means API working fine, now store this data into s3 bucket
boto3 is a pckg created by aws in order to programtically communicate with aws services
now dump entire json file into s3 , client putobject takes 3 params bucket, key (path wehere we sto store our data), body (spotify data json file)
update memory configuration to 5 mins
import json
import os
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import boto3

def lambda_handler(event, context):

    cilent_id = os.environ.get('client_id')
    client_secret = os.environ.get('client_secret')
    
    client_credentials_manager = SpotifyClientCredentials(client_id=cilent_id, client_secret=client_secret)
    sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)
    playlists = sp.user_playlists('spotify')
    
    playlist_link = "https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF?si=1333723a6eff4b7f"
    playlist_URI = playlist_link.split("/")[-1].split("?")[0]
    
    spotify_data = sp.playlist_tracks(playlist_URI)
    
    cilent = boto3.client('s3')
    
    cilent.put_object(
        Bucket="spotify-etl-project-satishkl1",
        Key="raw_data/to_processed/",
        Body=json.dumps(spotify_data)
        )
access denied error
when 2 services wnats to talk to each other or give permissions (need IAM role)
acess AWS act need permission emailid pwd we have IAM user for that
click on configurations-permissions  IAM > Roles > spotify_api_data_extract-role-ccz2o6tw
c basic permission, add permissions attach policies 'AmazonS3FullAccess'
test code again null response good 
now go to Amazon S3 > Buckets > spotify-etl-project-satishkl1 > raw_data/ > to_processed/ will see only /
so create filename with dateandtime, import datetiome test deploy

lambda_function.py

import json
import os
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import boto3
from datetime import datetime

def lambda_handler(event, context):

    cilent_id = os.environ.get('client_id')
    client_secret = os.environ.get('client_secret')
    
    client_credentials_manager = SpotifyClientCredentials(client_id=cilent_id, client_secret=client_secret)
    sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)
    playlists = sp.user_playlists('spotify')
    
    playlist_link = "https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF?si=1333723a6eff4b7f"
    playlist_URI = playlist_link.split("/")[-1].split("?")[0]
    
    spotify_data = sp.playlist_tracks(playlist_URI)
    
    cilent = boto3.client('s3')
    
    filename = "spotify_raw_" + str(datetime.now()) + ".json"
    
    cilent.put_object(
        Bucket="spotify-etl-project-satishkl1",
        Key="raw_data/to_processed/" + filename,
        Body=json.dumps(spotify_data)
        )
extraction done til here saved data onto s3 for every hour or min we extract data we will get new file or new data capture updated and hist data

noe process and transofmr data into defined folders album
copy and move data to processed folder once done, get only new to ensure not processing same data again and again

spotify_transformation_load_function create , attach existing role

list all the files and process one by one
import json
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    Bucket = "spotify-etl-project-satishkl1"
    Key = "raw_data/to_processed/"
    
    print(s3.list_objects(Bucket=Bucket, Prefix=Key)['Contents']))
'Key': 'raw_data/to_processed/'
    for file in s3.list_objects(Bucket=Bucket, Prefix=Key)['Contents']:
        print(file[Key])
list all files insode dictionary
store in var and read actual content from json file
if file key split is json, getobjects taks 2 paramas bucket filekey inside resp bosy actual data and pass to laod function
            jsonObject = json.loads(content.read())
            spotify_data.append(jsonObject)
            spotify_keys.append(file_key)
now we have files and data, create funcs loof for spotify data and call funcs

    spotify_data = []
    spotify_keys = []
    for file in s3.list_objects(Bucket=Bucket, Prefix=Key)['Contents']:
        file_key = file['Key']
        if file_key.split('.')[-1] == "json":
            response = s3.get_object(Bucket = Bucket, Key = file_key)
            content = response['Body']
            jsonObject = json.loads(content.read())
            spotify_data.append(jsonObject)
            spotify_keys.append(file_key)
            
    for data in spotify_data:
        album_list = album(data)
        artist_list = artist(data)
        song_list = songs(data)

        print(album_list) //test

now we have data in josn, convert into dataframe, do tranfs remove dups

    for data in spotify_data:
        album_list = album(data)
        artist_list = artist(data)
        song_list = songs(data)
        
        album_df = pd.DataFrame.from_dict(album_list)
        album_df = album_df.drop_duplicates(subset=['album_id'])
        
        artist_df = pd.DataFrame.from_dict(artist_list)
        artist_df = artist_df.drop_duplicates(subset=['artist_id'])
        
        #Song Dataframe
        song_df = pd.DataFrame.from_dict(song_list)
        
        album_df['release_date'] = pd.to_datetime(album_df['release_date'])
        song_df['song_added'] =  pd.to_datetime(song_df['song_added'])
create filename , convert above df into stringfile to csv function, then get value from it  and put into s3 object
        #Song Dataframe
        song_df = pd.DataFrame.from_dict(song_list)
        
        album_df['release_date'] = pd.to_datetime(album_df['release_date'])
        song_df['song_added'] =  pd.to_datetime(song_df['song_added'])
        
        songs_key = "transformed_data/songs_data/songs_transformed_" + str(datetime.now()) + ".csv"
        song_buffer=StringIO()
        song_df.to_csv(song_buffer, index=False)
        song_content = song_buffer.getvalue()
        s3.put_object(Bucket=Bucket, Key=songs_key, Body=song_content)
pandas error - add layer pandas , aws supports 
go to each folder Amazon S3 > Buckets > spotify-etl-project-satishkl1 > transformed_data/artist_data/ (will see processed transformed data)
now data is put on respective folders.
now moving data from toprocessed to processed folder ; first copying the data and deleting the data 

now automate , apply triggers
for lambda code extract add trigger schedule expression cron(0/5 0 * * ?) 
every 10 mins to_processed gets file in s3
apply another trigger
any obj created in to-processed .json (all object create events US East (N. Virginia) us-east-1
give lambda function acess to entore env because both lambdas to talk each other

now delete trigger s3 extract

create crawler now - we have files stored onto s3 bucket, read those files, get the schema info such as col names, what each adn every schema contains
once we do that glue crawler creates a table so that we can use it in athena
spotify_songs_crwaler -> add source songs folder bucket, choose or create IAM role, add DB spotify_db, create and run crawler
check table all cols crawled, go to athena and fire query
edit schema table and add manually if required in json, actions->edit table skipheadercnt

import json
import os
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials
import boto3
from datetime import datetime

def lambda_handler(event, context):

    cilent_id = os.environ.get('client_id')
    client_secret = os.environ.get('client_secret')
    
    client_credentials_manager = SpotifyClientCredentials(client_id=cilent_id, client_secret=client_secret)
    sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)
    playlists = sp.user_playlists('spotify')
    
    playlist_link = "https://open.spotify.com/playlist/37i9dQZEVXbNG2KDcFcKOF?si=1333723a6eff4b7f"
    playlist_URI = playlist_link.split("/")[-1].split("?")[0]
    
    spotify_data = sp.playlist_tracks(playlist_URI)
    
    cilent = boto3.client('s3')
    
    filename = "spotify_raw_" + str(datetime.now()) + ".json"
    
    cilent.put_object(
        Bucket="spotify-etl-project-satishkl1",
        Key="raw_data/to_processed/" + filename,
        Body=json.dumps(spotify_data)
        )


import json
import boto3
from datetime import datetime
from io import StringIO
import pandas as pd

def album(data):
    album_list = []
    for row in data['items']:
        album_id = row['track']['album']['id']
        album_name = row['track']['album']['name']
        album_release_date = row['track']['album']['release_date']
        album_total_tracks = row['track']['album']['total_tracks']
        album_url = row['track']['album']['external_urls']['spotify']
        album_element = {'album_id':album_id,'name':album_name,'release_date':album_release_date,
                            'total_tracks':album_total_tracks,'url':album_url}
        album_list.append(album_element)
    return album_list
    
def artist(data):
    artist_list = []
    for row in data['items']:
        for key, value in row.items():
            if key == "track":
                for artist in value['artists']:
                    artist_dict = {'artist_id':artist['id'], 'artist_name':artist['name'], 'external_url': artist['href']}
                    artist_list.append(artist_dict)
    return artist_list
    
def songs(data):
    song_list = []
    for row in data['items']:
        song_id = row['track']['id']
        song_name = row['track']['name']
        song_duration = row['track']['duration_ms']
        song_url = row['track']['external_urls']['spotify']
        song_popularity = row['track']['popularity']
        song_added = row['added_at']
        album_id = row['track']['album']['id']
        artist_id = row['track']['album']['artists'][0]['id']
        song_element = {'song_id':song_id,'song_name':song_name,'duration_ms':song_duration,'url':song_url,
                        'popularity':song_popularity,'song_added':song_added,'album_id':album_id,
                        'artist_id':artist_id
                       }
        song_list.append(song_element)
        
    return song_list


def lambda_handler(event, context):
    s3 = boto3.client('s3')
    Bucket = "spotify-etl-project-satishkl1"
    Key = "raw_data/to_processed/"
    
    spotify_data = []
    spotify_keys = []
    
    for file in s3.list_objects(Bucket=Bucket, Prefix=Key)['Contents']:
        file_key = file['Key']
        if file_key.split('.')[-1] == "json":
            response = s3.get_object(Bucket = Bucket, Key = file_key)
            content = response['Body']
            jsonObject = json.loads(content.read())
            spotify_data.append(jsonObject)
            spotify_keys.append(file_key)
            
    for data in spotify_data:
        album_list = album(data)
        artist_list = artist(data)
        song_list = songs(data)
        
        album_df = pd.DataFrame.from_dict(album_list)
        album_df = album_df.drop_duplicates(subset=['album_id'])
        
        artist_df = pd.DataFrame.from_dict(artist_list)
        artist_df = artist_df.drop_duplicates(subset=['artist_id'])
        
        #Song Dataframe
        song_df = pd.DataFrame.from_dict(song_list)
        
        album_df['release_date'] = pd.to_datetime(album_df['release_date'])
        song_df['song_added'] =  pd.to_datetime(song_df['song_added'])
        
        songs_key = "transformed_data/songs_data/songs_transformed_" + str(datetime.now()) + ".csv"
        song_buffer=StringIO()
        song_df.to_csv(song_buffer, index=False)
        song_content = song_buffer.getvalue()
        s3.put_object(Bucket=Bucket, Key=songs_key, Body=song_content)
        
        album_key = "transformed_data/album_data/album_transformed_" + str(datetime.now()) + ".csv"
        album_buffer=StringIO()
        album_df.to_csv(album_buffer, index=False)
        album_content = album_buffer.getvalue()
        s3.put_object(Bucket=Bucket, Key=album_key, Body=album_content)
        
        artist_key = "transformed_data/artist_data/artist_transformed_" + str(datetime.now()) + ".csv"
        artist_buffer=StringIO()
        artist_df.to_csv(artist_buffer, index=False)
        artist_content = artist_buffer.getvalue()
        s3.put_object(Bucket=Bucket, Key=artist_key, Body=artist_content)
        
    s3_resource = boto3.resource('s3')
    for key in spotify_keys:
        copy_source = {
            'Bucket': Bucket,
            'Key': key
        }
    s3_resource.meta.client.copy(copy_source, Bucket, 'raw_data/processed/' + key.split("/")[-1])    
    s3_resource.Object(Bucket, key).delete()



