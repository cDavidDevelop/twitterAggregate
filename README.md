# twitterAggregate
Aggregate Tweets to Slack channel for content filtering and meta data analysis

## Summary
We want to gather media (photos and videos) of viral content with the ability to easily and manually filter it along the way.

## The Project

### From the User's Perspective

1. Show me very popular Tweets that include media (photos / videos) that we think could be viral in the future, via a dedicated Slack Channel.

2. Allow me to click "Keep" or "Discard" buttons on each Slack notification to keep or discard the content for later.

3. Provider a simple admin Web UI
  a. Set the number of maximum Slack's sent per day.
  b. View saved content
  c. Enable tagging saved content
  d. Enable deleting saved content.
  e. Enable searching saved content by tags.
  f. Use Vue, Angular, or react for front-end
  

### PreText Storage
Database storage:
CREATE TABLE CONFIG
  Columns: APPLICATION, KEY, VAL
insert into CONFIG(application, key, val) values ("twitter", "Max Messages Per Day", "60")

CREATE TABLE TWITTER...
  Columns: insert_dttm, url, original_poster, media, tags
  or
  Columns: insert_dttm, json_structure, keep_flag, discard_flag, hash_tags
  

### Farming tweets, and posting to Slack : Part 1: Grabbing Very popular Tweets

Twitter API Accessability: Java, JavaScript, Ruby, Python, C#, Go, R
C# Starter Code Searching for content with hashtags https://developer.twitter.com/
```
using System;
using System.Net.Http;
using Newtonsoft.Json;

namespace TwitterAPIExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Replace these values with your own Twitter API keys and secrets
            string consumerKey = "YOUR_CONSUMER_KEY";
            string consumerSecret = "YOUR_CONSUMER_SECRET";
            string accessToken = "YOUR_ACCESS_TOKEN";
            string accessTokenSecret = "YOUR_ACCESS_TOKEN_SECRET";

            // Set up the HttpClient
            var client = new HttpClient();
            client.DefaultRequestHeaders.Add("Authorization", "Bearer " + GetBearerToken(consumerKey, consumerSecret));
            client.DefaultRequestHeaders.Add("User-Agent", "YOUR_APP_NAME");

            // Set the query parameters
            var queryString = "q=%23YOUR_HASHTAG&result_type=recent&count=100";

            // Send the request and get the response
            var response = client.GetAsync("https://api.twitter.com/2/tweets/search/recent?" + queryString).Result;
            var json = response.Content.ReadAsStringAsync().Result;

            // Deserialize the JSON into a list of tweets
            var tweets = JsonConvert.DeserializeObject<List<Tweet>>(json);

            // Print out the text of each tweet
            foreach (var tweet in tweets)
            {
                Console.WriteLine(tweet.Text);
            }
        }

        // This method gets a bearer token that can be used to authenticate API requests
        static string GetBearerToken(string consumerKey, string consumerSecret)
        {
            // Set up the HttpClient
            var client = new HttpClient();
            client.DefaultRequestHeaders.Add("User-Agent", "YOUR_APP_NAME");

            // Set the request parameters
            var requestParams = new Dictionary<string, string>
            {
                { "grant_type", "client_credentials" }
            };
            var requestContent = new FormUrlEncodedContent(requestParams);

            // Set the request URL and authorization headers
            var requestUrl = "https://api.twitter.com/oauth2/token";
            var authorizationHeader = Convert.ToBase64String(Encoding.UTF8.GetBytes(Uri.EscapeDataString(consumerKey) + ":" + Uri.EscapeDataString(consumerSecret)));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", authorizationHeader);

            // Send the request and get the response
            var response = client.PostAsync(requestUrl, requestContent).Result;
            var json = response.Content.ReadAsStringAsync().Result;

            // Deserialize the JSON and get the bearer token
            var tokenResponse = JsonConvert.DeserializeObject<TokenResponse>(json);
            return tokenResponse.AccessToken;
        }
    }

[ . . . Process URLs to download media to Database, or File location Stored in Database] 
```

### Farming Tweets, and posting to Slack : Part 2: Slack
Slack API website (https://api.slack.com/)

### 2. Slack channel usage.
https://api.slack.com/messaging/webhooks
https://api.slack.com/messaging/interactivity
"Keep" or "Discard" buttons on each Slack notification

 
#### Vue
```
<template>
  <div id="app">
    <form>
      <label for="max-messages">Maximum messages per day:</label>
      <input id="max-messages" type="text" v-model="maxMessages">
    </form>
    <div id="content-container">
      <input type="text" v-model="newHashtag" placeholder="Add a hashtag">
      <button @click="addHashtag">Add</button>
      <button @click="deleteContent">Delete</button>
      <ul>
        <li v-for="hashtag in hashtags" :key="hashtag">{{ hashtag }}</li>
      </ul>
    </div>
    <input type="text" v-model="search" placeholder="Search">
  </div>
</template>

<script>
export default {
  name: 'app',
  data() {
    return {
      maxMessages: '',
      newHashtag: '',
      hashtags: [],
      search: ''
    }
  },
  methods: {
    addHashtag() {
      this.hashtags.push(this.newHashtag)
      this.newHashtag = ''
    },
    deleteContent() {
      this.hashtags = []
    }
  }
}
</script>
```

#### Angular
```
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div id="app">
      <form>
        <label for="max-messages">Maximum messages per day:</label>
        <input id="max-messages" type="text" [(ngModel)]="maxMessages" name="maxMessages">
      </form>
      <div id="content-container">
        <input type="text" [(ngModel)]="newHashtag" name="newHashtag" placeholder="Add a hashtag">
        <button (click)="addHashtag()">Add</button>
        <button (click)="deleteContent()">Delete</button>
        <ul>
          <li *ngFor="let hashtag of hashtags">{{ hashtag }}</li>
        </ul>
      </div>
      <input type="text" [(ngModel)]="search" name="search" placeholder="Search">
    </div>
  `,
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  maxMessages = '';
  newHashtag = '';
  hashtags = [];
  search = '';

  addHashtag() {
    this.hashtags.push(this.newHashtag);
    this.newHashtag = '';
  }

  deleteContent() {
    this.hashtags = [];
  }
}

```

#### React
```
import React, { useState } from 'react';

function App() {
  const [maxMessages, setMaxMessages] = useState('');
  const [newHashtag, setNewHashtag] = useState('');
  const [hashtags, setHashtags] = useState([]);
  const [search, setSearch] = useState('');

  const handleAddHashtag = () => {
    setHashtags([...hashtags, newHashtag]);
    setNewHashtag('');
  }

  const handleDeleteContent = () => {
    setHashtags([]);
  }

  return (
    <div id="app">
      <form>
        <label htmlFor="max-messages">Maximum messages per day:</label>
        <input 
          id="max-messages" 
          type="text" 
          value={maxMessages} 
          onChange={e => setMaxMessages(e.target.value)}
        />
      </form>
      <div id="content-container">
        <input 
          type="text" 
          value={newHashtag} 
          onChange={e => setNewHashtag(e.target.value)}
          placeholder="Add a hashtag"
        />
        <button onClick={handleAddHashtag}>Add</button>
        <button onClick={handleDeleteContent}>Delete</button>
        <ul>
          {hashtags.map(hashtag => (
            <li key={hashtag}>{hashtag}</li>
          ))}
        </ul>
      </div>
      <input 
        type="text" 
        value={search} 
        onChange={e => set

```

