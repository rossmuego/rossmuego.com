---
title:  "Social Media Site"
date:   2017-11-01 00:00:00
categories: Web
tags: PHP
      SQL
      HTML
      JavaScript
---
During my last year of Secondary School I undertook my Coursework Project/Dissertation. I had the entire year to complete this task and was required to demonstrate a number of skills. I chose to create a Social Network focussed around the distribution and sharing of music related videos embedded from [YouTube][youtube-link].

To complete this task I used a range of languages and skills. This included HTML, CSS, PHP, SQL, jQuery and Javascript. The project was required to go through the complete design process including full documentation, designing and testing. Surveys were carried out to gather user requirements.

To check out the full project and solution, [click here][github-repo] to see the GitHub repository.


## Overview

The site allowed users to sign up for the website with a unique username along with a password. They could follow people and people could follow them. It allowed users to search for one another using their usernames. Users could customise their profiles with their social media information along with a profile picture to identify the user. Users could submit videos via a submission page which would require them to enter a URL to the video, the name and artist. If the video had already been published it would direct the user to the video page. Users had the option to rate videos via a 'thumbs up/thumbs down' system and they also had the ability to comment on videos.

## Login/Registration

The users were promoted to login to the site or signup when the first arrived on the site. This information would ultimately be stored in a 'Users' table in a database. It requires the user to enter their name, a username, email a password and a password confirmation.

**Login PHP**

```php
<?php
include_once('db_conx.php');

    session_start();

    $email = $_POST['email'];
    $password = $_POST['password'];

    echo $email;
    echo $password;

        $sql = "SELECT * FROM users WHERE email='$email' AND password='$password'";
        $queryLogin = mysqli_query($conn, $sql);
        $loginCheck = mysqli_num_rows($queryLogin);

    if($loginCheck==0){
			echo 'error';
            $_SESSION['login'] = false;
        } else {
            echo 'success';
            $_SESSION['login'] = true;
        }
?>
```

**Registration PHP**

```php
<?php

include_once('db_conx.php');

session_start();

    $name = $_POST['name'];
    $email = $_POST['email'];
    $password = $_POST['password'];
    $username = $_POST['username'];

// Check if e-mail address syntax is valid or not

    $email = filter_var($email, FILTER_SANITIZE_EMAIL); // Sanitizing email
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)){
        echo "Invalid Email.";
    } else {
        $result = mysqli_query($conn, "SELECT * FROM users WHERE email='$email' OR username='$username'");
        $data = mysqli_num_rows($result);

        if(($data)==0){
            $query = mysqli_query($conn, "INSERT INTO profiles(profile_pic) VALUES ('default.png')");
            $query = mysqli_query($conn, "INSERT INTO users(name, email, username, password) VALUES ('$name', '$email', '$username', '$password')"); // Insert query
            echo "registerSuccess";
        } else{
            echo "This user is already registered, Please try another email/username.";
        }
    }
?>
```

## Submitting a Post

Users have the ability to submit posts via the submission page. The submission page requires users to enter a URL for the video they wish to submit, along with the name of the submission and the artist. If the video has already been submitted, the user will be directed to the page corresponding with that video where they will be able to comment and like/dislike the video.

![SubmitPost]({{ "/images/posts/social-media/submitvid.png"}})

**Submission PHP**

```php
<?php

	session_start();

	include_once('db_conx.php');
  include_once('header.php');

  $songName = $_POST['songName'];
  $artist = $_POST['artist'];
  $songPOST = $_POST['songLink'];
	$songQuery = false;
	$user = $_SESSION['user_id'];

  $getID = $songPOST;
	parse_str( parse_url( $getID, PHP_URL_QUERY ), $songShort );
	$songID = $songShort['v'];    

	$sql = "SELECT * FROM submissions WHERE songID ='$songID'";
	$result = $conn->query($sql);

	if ($result->num_rows > 0) {
    // output data of each row
		$submitStatus = "This song has already been posted!";
	} else {
	    $songQuery = true;
	}

	if($songQuery) {
		$sql = "INSERT INTO submissions (songName, artist, songID, submitter) VALUES ('$songName', '$artist', '$songID', '$user')";

		if ($conn->query($sql) === true) {
		    $submitStatus = "Song Submitted!";
		} else {
		    echo "Error: " . $sql . "<br>" . $conn->error;
		}
		$conn->close();
	}
?>
```

## Searching for Users

Along the header of the site is the ability for users to search for one another via usernames. The search results appear in realtime and display the users username along with their profile picture. In order to do this I used a JavaScript script paired with PHP to in realtime query the database.

**JavaScript**

```js
$(document).ready(function() {
     $("#search_input").keyup(function(){
         var search_input = $(this).val();
         var dataString = 'keyword='+ search_input;

         if(search_input.length>0){

            //AJAX POST
            $.ajax({
                 type: "POST",
                 url: "searchbar.php",
                 data: dataString,
                success: function(response){
                     $('#searchresultdata').html(response).show();
                 }
             });
         } else {
			$("#searchresultdata").hide();
	 }
     });
});
```
**PHP**

```php
<?php
    include('db_conx.php');

    if(isset($_POST['keyword'])){
        $keyword = trim($_POST['keyword']);
        $keyword = mysqli_real_escape_string($conn, $keyword);
        $query = "SELECT user_id, username FROM users WHERE username LIKE '%$keyword%' LIMIT 5"; //MUST BEGIN WITH $KEYWORD

        //echo $query;
        $result = mysqli_query($conn,$query);

        if($result){
            if(mysqli_affected_rows($conn)!=0){

                while($row = mysqli_fetch_array($result, MYSQLI_ASSOC)){

                    $userID = $row['user_id'];

                    $sql = "SELECT profile_pic FROM users WHERE user_id = $userID";
                    $result1 = mysqli_query($conn,$sql);

                    if($result1){
                        if(mysqli_affected_rows($conn)!=0){
                            while($row1 = mysqli_fetch_array($result1, MYSQLI_ASSOC)){
                                $profilePicSearch = $row1['profile_pic'];
                            }
                    }
                    }

                    $usernameDisplay = $row['username'];

                    echo '<a href="profile.php?id='.$userID.'"><div class="userSearchDisplay"><img src="../images/profiles/'.$profilePicSearch.'" alt="Profile Pic" class="profilePic" style="width: 50px; height: 50px;">'.$usernameDisplay.'</div></a></br>';
                }
            }else {
                echo '<div class="userSearchDisplay">No Results for :"'.$_POST['keyword'].'"</div>';
            }
        }
    }else {
        echo 'Parameter Missing';
    }
?>
```

## Following a User

In order to follow and unfollow users in realtime I once again used PHP along with JavaScript. I created a table within my database recording which users followed which and every time there was a follow/unfollow action it would be updated in realtime and dynamically changing the follow/unfollow buttons state.

**PHP**

```php
<?php

  include_once('db_conx.php');
  session_start();

  $action = $_POST['action'];
  $followID = $_POST['id'];
  $follower = $_SESSION['user_id'];

  if($action == "follow"){

      $sql = "INSERT INTO `following`(`follower_id`, `following_id`, `followed`) VALUES ($follower, $followID, 1)";

      if ($conn->query($sql) === true) {
          echo '<a class="btn disabled">Following</a>';
        } else {
            echo "Error: " . $sql . "<br>" . $conn->error;
          }
  } elseif($action == "unfollow"){
      $sql = "DELETE FROM following WHERE follower_id = $follower AND following_id = $followID";

      if ($conn->query($sql) === true) {
          echo '<a class="btn disabled">Following</a>';
      } else {
          echo "Error: " . $sql . "<br>" . $conn->error;
      }
  }
?>
```

**JavaScript for follow**

```js
function followUser(clicked_id){

  //the main ajax request  
  $.ajax({  
    type: "POST",  
    data: "action=follow&id="+clicked_id,  
    url: "follow.php",  
    success: function(msg) {
        $("#"+clicked_id).removeClass("waves-effect waves-light btn");
        $("#"+clicked_id).addClass("btn green");
        $("#"+clicked_id).html("Unfollow");
        $("#"+clicked_id).attr("onclick","unfollowUser(this.id)");
    }  
  });  
};
```
![FollowUser]({{ "/images/posts/social-media/followuser.png"}})

**JavaScript for unfollow**

```js
function unfollowUser(clicked_id){

  //the main ajax request  
  $.ajax({  
    type: "POST",  
    data: "action=unfollow&id="+clicked_id,  
    url: "follow.php",  
    success: function(msg) {
        $("#"+clicked_id).removeClass("btn green");
        $("#"+clicked_id).addClass("waves-effect waves-light btn");
        $("#"+clicked_id).html("Follow");
        $("#"+clicked_id).attr("onclick","followUser(this.id)");
    }  
  });  
};
```
![UnfollowUser]({{ "/images/posts/social-media/unfollowuser.png"}})


## Rating System

Users are able to rate the videos posted by users using the 2 thumbs beside each post. One arrow corresponds to a positive vote whilst the other corresponds to a negative vote. The difference between the 2 vote types is displayed beside the video (i.e. 100 votes up, 50 votes down = displaying 50 votes). The user can only vote on the video once.

![UnfollowUser]({{ "/images/posts/social-media/votes.png"}})

**PHP**

```php
<?php

    include_once('db_conx.php');

    session_start();

        $user = $_SESSION['email'];
        $songID = $_POST['id'];
        $voteValid = "";

        $result = mysqli_query($conn, "SELECT * FROM votes WHERE songID='$songID' AND user='$user'");

        if(mysqli_num_rows($result)){
            $voteValid = 'false';
        } else {
            $voteValid = 'true';
        }

    $action = $_POST['action'];

    if($voteValid == 'true'){

        if($action == "vote_up") {

            $songID = $_POST['id'];

            $sql = "UPDATE submissions SET votes_up = votes_up+1 WHERE songID = '$songID'";
            $query = mysqli_query($conn, $sql);

            $sql = "UPDATE submissions SET vote_diff = votes_up - votes_down WHERE songID = '$songID'";
            $query = mysqli_query($conn, $sql);

            //EXTRACT WHILE LOOP DIFFERENCE BETWEEN VOTES

            $sql = "SELECT vote_diff FROM submissions WHERE songID = '$songID'";
            $voteDiff = mysqli_query($conn, $sql);

            while($voteDiffExtract = mysqli_fetch_array($voteDiff, MYSQLI_ASSOC)) {
                $voteDiffValue = $voteDiffExtract['vote_diff'];
            }

            $query = mysqli_query($conn, "INSERT INTO votes (user, songID, voted) VALUES ('$user', '$songID', '1')"); // Insert query

            echo $voteDiffValue;

            exit();

        } elseif($action == "vote_down") {


            $songID = $_POST['id'];

            $sql = "UPDATE submissions SET votes_down = votes_down+1 WHERE songID = '$songID'";
            $query = mysqli_query($conn, $sql);

            $sql = "UPDATE submissions SET vote_diff = votes_up - votes_down WHERE songID = '$songID'";
            $query = mysqli_query($conn, $sql);

            $sql = "SELECT vote_diff FROM submissions WHERE songID = '$songID'";
            $voteDiff = mysqli_query($conn, $sql);

            while($voteDiffExtract = mysqli_fetch_array($voteDiff, MYSQLI_ASSOC)) {
               $voteDiffValue = $voteDiffExtract['vote_diff'];
           }

           $query = mysqli_query($conn, "INSERT INTO votes (user, songID, voted) VALUES ('$user', '$songID', '2')"); // Insert query

           echo $voteDiffValue;

            exit();
        }
    } else {
        echo 'You have already voted.';
    }
?>
```

**JavaScript**

```js
function votesUp(clicked_id) {

  //the main ajax request  
  $.ajax({  
    type: "POST",  
    data: "action=vote_up&id="+clicked_id,  
    url: "votes.php",  
    success: function(msg) {
      console.log(msg);
      if (msg.indexOf("You have already voted.") > -1) {
                    alert(msg);//code
                } else {
        $('#displayCount'+clicked_id+'top').html(msg);
        $('#displayCount'+clicked_id+'new').html(msg);
      }
    }  
  });   
};

function downVote(clicked_id){

  //the main ajax request  
  $.ajax({  
    type: "POST",  
    data: "action=vote_down&id="+clicked_id,  
    url: "votes.php",  
    success: function(msg) {
      if (msg.indexOf("You have already voted.") > -1) {
                    alert(msg);//code
                } else {
        $('#displayCount'+clicked_id+'top').html(msg);
        $('#displayCount'+clicked_id+'new').html(msg);
      }
    }  
  });  
};

```

The functions for votesUp and downVote work by taking the id from the div the video is embedded in which corresponds to the unique id the video is assigned when it is uploaded to the site.


[youtube-link]:   www.youtube.com
[github-repo]: https://github.com/rossmuego/SocialMediaSite
