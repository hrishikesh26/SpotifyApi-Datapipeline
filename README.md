# SpotifyApi - Data collection pipeline using Apache-Airflow service and AWS services🪁 

### Architecture ↘️

  ![image](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/f16d5070-25b1-4108-bd9b-507e73b0aa18)  
  
  
  # 💡 What is Apache-Airflow.
  - Apache Airflow is an open-source platform used for orchestrating and scheduling complex data workflows. It allows users to define, schedule, and monitor workflows as directed acyclic graphs (DAGs). Airflow provides a flexible and scalable framework for managing data pipelines, allowing users to easily define dependencies, execute tasks, and monitor their progress. It supports a wide range of integrations and provides a web-based UI for managing and monitoring workflows. Overall, Apache Airflow simplifies the process of managing and automating data pipelines in a reliable and scalable manner. <br>

  # 💡What is SpotifyAPI
  - The Spotify API is a set of web-based endpoints and tools provided by Spotify, the popular music streaming platform. It allows developers to access and interact with Spotify's vast catalog of music, user data, playlists, and more programmatically. With the Spotify API, developers can build applications that integrate with Spotify's features, such as searching for music, retrieving artist and album information, creating and modifying playlists, and controlling music playback. It enables developers to leverage Spotify's music data and functionality to enhance their own applications and create unique music-related experiences for users. <br>


## Overview 📔

- This project showcases a comprehensive solution for data collection and sorting from the Spotify API, leveraging the powerful Airflow service hosted on an AWS EC2 T3 Medium instance. By combining the capabilities of the Spotify API, Airflow, and AWS services, this project provides a seamless workflow for gathering and organizing Spotify data.

- Using Airflow, an open-source platform for workflow automation, a series of tasks are set up to periodically retrieve data from the Spotify API. These tasks are orchestrated and scheduled within Airflow, ensuring efficient and reliable data collection.

- The project utilizes an AWS EC2 T3 Medium instance for hosting the Airflow service. This instance type offers a balance of compute power and cost-effectiveness, making it ideal for running the data collection pipeline.

- To facilitate efficient data storage and organization, the project integrates AWS S3 buckets. Python code, written within the Airflow service, is employed to sort and store the collected Spotify data in designated S3 buckets. This approach enables easy access, retrieval, and further analysis of your spotify data.

- By leveraging the Spotify API, Airflow, and AWS services, this project provides a robust and scalable infrastructure for collecting and organizing Spotify data. The modular design and automation capabilities of the workflow allow for easy customization and extension to meet specific requirements, making it a valuable resource for various applications, such as music analytics, recommendation systems, and research projects.



# Initial setup for hosting airflow service : - 
- Login to your aws account. 
- Launch an `T3 medium` ec2 instance with `ubuntu` as your OS. In security groups allow SSH access to your instance. 
- Open putty and login through your `public ip` address. Select your ppk file as a key by navigating through 'credentials' sections in putty's console. 
- Use `ubuntu` as sure login user in the terminal. 
- Initially you need to prepare your machine. For that run the following commands : -
  - `sudo apt-get update`
  - `sudo apt install python3-pip`
  - `sudo pip isntall apache-airflow`
  - `sudo pip install datetime`
  - `sudo pip install s3fs`
  - `sudo pip install pandas` 
  - `sudo pip install spotipy` 
 
# Now that your machine is ready ✔️, Let's proceed step by step -
- After executing these commands type `airflow` to check if the airflow service is installed correctly. 
- Now to run airflow service use `airflow standalone` command. This will start your airflow service on your ec2 instance.
- Search for your login credentials given by ariflow service in the terminal. The login credentials would look like this - 

![airflow server login credentials](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/c155d3f7-adcd-45c8-af0d-3f6a92a116ed)

- Note down these login credentials, and go to your ec2 instance on aws. Click and copy your `DNS ip` and add `:8080` at the end of DNS ip address. 



![create instance](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/3e314b6b-1e4d-435a-a76b-38a545527bff)

- edited url - 
![edit url for airflow server](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/5061e6e0-376b-441c-b287-d00db7dee2bd)


- Enter the airflow login credentials. 

![ariflow login page](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/b2340014-03a6-46ae-b950-c0f0e76519c3)

## Now that you have successfully logged into your airflow service, you also need to make provision for saving the data received from spotify api. <br>
- For this, create an AWS s3 bucket and give IAM role of full access of s3 to this EC2 instance. This will allow your instance to make changes ( create / delete / access ) in the s3 bucket.

  - **create s3 bucket** <br>
  
    ![create s3 bucket to store spotify api data](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/41b13535-4d4b-4051-b71d-d559eb5be559)

  - **Create IAM role** <br>


  ![add role for ec2 for s3](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/d2b11c2e-ca25-4ce6-aeaa-971f557b9ad1)

  - **Assign the role to ec2 instance.** <br> 

  ![assign role to ec2](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/efa59914-9664-4abc-920b-cb536521be2f)
  
  
  
## Making changes in ec2 instance terminal - 
- Use `ls` command to view the directories under your home directory. "airflow" will be displayed
- Go to Airflow folder, use `cd airflow`.
- Again use `ls` and navigate to airflow.cfg file. Here you need to make changes into the file, for that use `sudo vi ariflow.cfg` command to open and edit the file. 
  - in the file change the dags_folder object content from "/home/ubuntu/airflow/dags" to "/home/ubuntu/airflow/spotify_dag". <br>
  
    ![image](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/5803f1ac-32ef-48d9-80f9-6bd04e2dda90)
    
- Now, create a directory using `sudo mkdir spotify_dag` command.
- Navigate into this folder - `cd spotify_dag`. 
- Here you need to insert a python file. Use `sudo vi spotify_etl.py` to create folder in **spotify_dag** folder. <br> ( write python code to extract & transform data using spotify api & upload the datasets to s3 bucket ).


    ```
    import spotipy
    import pandas as pd 
    from spotipy.oauth2 import SpotifyClientCredentials
    from datetime import datetime
    import s3fs 

    def run_spotify_etl():

        client_id = "b8db1f5e394149a69d57283ea2e75761"
        client_secret = "93425fac35e94d59b6a90f6714443668"
    # Provide details for authentication to extract data from spotify
        client_credentials_mgr = SpotifyClientCredentials(client_id=client_id,
                                                          client_secret=client_secret)


        # Create an object to provides authorization to access data to extract
        sp = spotipy.Spotify(client_credentials_manager = client_credentials_mgr)

        playlist_link = "https://open.spotify.com/playlist/7oFbQJn36KPaMHEwgT1XF4"
        playlist_URI = playlist_link.split('/')[-1]

        # To extract all the info related to tracks
        data = sp.playlist_tracks(playlist_URI)

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
        album_df = pd.DataFrame.from_dict(album_list)

        artist_list = []
        for row in data['items']:
            for key,value in row.items():
                if key == 'track':
                    for artist in value['artists']:
                        artist_dict = {'artist_id':artist['id'],'artist_name':artist['name'],'externam_url':artist['external_urls']['spotify']}
                        artist_list.append(artist_dict)
        artist_df = pd.DataFrame.from_dict(artist_list)

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
            song_dict = {'song_id':song_id,'song_name':song_name,'duration_ms':song_duration,'url':song_url,
                        'popularity':song_popularity,'song_added':song_added,'album_id':album_id,'artist_id':artist_id}
            song_list.append(song_dict)
        song_df = pd.DataFrame.from_dict(song_list)


        album_df = album_df.drop_duplicates(subset='album_id')
        artist_df = artist_df.drop_duplicates(subset='artist_id')
        song_df = song_df.drop_duplicates(subset='song_id')

        album_df.to_csv("s3://spotifydatabucket/album.csv")    # use your bucket name here
        artist_df.to_csv("s3://spotifydatabucket/artist_df.csv")
        song_df.to_csv("s3://spotifydatabucket/song_df.csv")
    ```


- Create another pythone file in the same directory `sudo vi spotify_dag.py` (write python code to create dag for the spotify etl process).
      ```
        from datetime import timedelta
        from airflow import DAG
        from airflow.operators.python_operator import PythonOperator
        from airflow.utils.dates import days_ago
        from datetime import datetime
        from spotify_etl import run_spotify_etl

        default_args = {
            'owner': 'airflow',
            'depends_on_past': False,
            'start_date': datetime(2020, 11, 8),
            'email': ['airflow@example.com'],
            'email_on_failure': False,
            'email_on_retry': False,
            'retries': 1,
            'retry_delay': timedelta(minutes=1)
        }

        dag = DAG(
            'spotify_dag',
            default_args=default_args,
            description='Our first DAG with ETL process!',
            schedule_interval=timedelta(days=1),
        )

        run_etl = PythonOperator(
            task_id='complete_spotify_etl',
            python_callable=run_spotify_etl,
            dag=dag, 
        )

        run_etl
      ```


## Now go to the airflow server
- You were already logged in so you need to spot your folder you created above. Search for **spotify_dag**, it will be seen as below 👇
    
    ![image](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/6ccf40d2-3c18-4175-8032-b4c2d5e08982)


- After clicking on "Spotify_dag" dag you can view your etl code as shown below. 

    ![image](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/fd7d945f-3a25-4cf7-af0c-6a74bb860efa)
    

- Now, run the dag using trigger DAG option :

    ![image](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/f80d3078-3527-455f-aa15-f669aed0bdda)

    **Dark green border shows that your dag was successfully executed*
    
- Now if you navigate back to your s3 bucket on AWS account you can see that the desired data is extracted from SpotifyAPI in the form of csv. 

    ![s3 bucket contents after executing](https://github.com/hrishikesh26/SpotifyApi-Datapipeline/assets/94166344/76f963b5-da7c-495e-afd9-927de6c28e52)

    
➡️ With the help of these csv's you can furhter do data analytics using AWS services such as AWS Quicksight & AWS Athena (quering service). 

**Note : Please refer to the ipynb notebook included in the repository to view the schema of data provide by SpotifyAPI**
