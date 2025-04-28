<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Social Media App</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; background-color: #f0f0f0; }
    .container { max-width: 800px; margin: auto; background-color: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); }
    .postItem { border: 1px solid #ddd; padding: 15px; margin-bottom: 10px; border-radius: 8px; }
    .postItem button { margin-right: 10px; }
    .postItem input { width: 100%; padding: 8px; margin-top: 5px; }
    .comments { margin-top: 10px; padding-left: 20px; }
    .search-box { width: 100%; padding: 8px; margin-bottom: 20px; }
    .auth-section { margin-bottom: 20px; }
    .auth-section input { width: 100%; padding: 8px; margin-bottom: 10px; }
    .auth-section button { width: 100%; padding: 10px; background-color: #007BFF; color: white; border: none; cursor: pointer; }
    .auth-section button:hover { background-color: #0056b3; }
  </style>
</head>
<body>

<div class="container">
  <!-- Authentication Section -->
  <div id="auth-section" class="auth-section">
    <h2>Login / Sign Up</h2>
    <input type="text" id="username" placeholder="Username" required><br>
    <input type="password" id="password" placeholder="Password" required><br>
    <button onclick="signUp()">Sign Up</button>
    <button onclick="logIn()">Log In</button>
    <p id="auth-error" style="color:red;"></p>
  </div>

  <!-- Post Creation Section (Visible after login) -->
  <div id="post-section" style="display:none;">
    <h2>Create Post:</h2>
    <textarea id="postContent" rows="4" placeholder="What's on your mind?"></textarea><br><br>
    <button onclick="createPost()">Post</button>
  </div>

  <!-- Search Box -->
  <input type="text" id="search" class="search-box" placeholder="Search posts..." onkeyup="searchPosts()" style="display:none;">
  
  <!-- Posts Section -->
  <div id="posts"></div>
</div>

<script>
// Simulating user authentication and posts with localStorage
let users = JSON.parse(localStorage.getItem('users')) || [];
let posts = JSON.parse(localStorage.getItem('posts')) || [];
let loggedInUser = JSON.parse(localStorage.getItem('loggedInUser')) || null;

// Display posts when the user is logged in
function displayPosts() {
  const postsContainer = document.getElementById("posts");
  postsContainer.innerHTML = '';

  posts.forEach((post, index) => {
    const postDiv = document.createElement("div");
    postDiv.classList.add("postItem");

    postDiv.innerHTML = `
      <p><strong>${post.user}</strong>: ${post.content}</p>
      <button onclick="likePost(${index})">Like (${post.likes})</button>
      <button onclick="dislikePost(${index})">Dislike (${post.dislikes})</button>
      <button onclick="sharePost(${index})">Share</button>
      <button onclick="followUser('${post.user}')">Follow ${post.user}</button>
      <div class="comments">
        <input type="text" id="comment-input-${index}" placeholder="Write a comment..." onkeypress="addComment(event, ${index})">
        <div id="comments-${index}">${post.comments.join('<br>')}</div>
      </div>
    `;
    postsContainer.appendChild(postDiv);
  });
}

// Sign Up Function
function signUp() {
  const username = document.getElementById("username").value;
  const password = document.getElementById("password").value;

  if (!username || !password) {
    document.getElementById("auth-error").innerText = "Please enter both username and password.";
    return;
  }

  if (users.some(user => user.username === username)) {
    document.getElementById("auth-error").innerText = "Username already exists!";
    return;
  }

  users.push({ username, password });
  localStorage.setItem("users", JSON.stringify(users));
  alert("Sign Up successful! You can now log in.");
}

// Log In Function
function logIn() {
  const username = document.getElementById("username").value;
  const password = document.getElementById("password").value;

  const user = users.find(user => user.username === username && user.password === password);
  if (user) {
    loggedInUser = user;
    localStorage.setItem("loggedInUser", JSON.stringify(loggedInUser));
    document.getElementById("auth-section").style.display = "none";
    document.getElementById("post-section").style.display = "block";
    document.getElementById("search").style.display = "block";
    displayPosts();
  } else {
    document.getElementById("auth-error").innerText = "Invalid credentials!";
  }
}

// Create Post Function
function createPost() {
  const content = document.getElementById("postContent").value;

  if (!content) {
    alert("Please write something to post.");
    return;
  }

  const newPost = {
    user: loggedInUser.username,
    content,
    likes: 0,
    dislikes: 0,
    comments: []
  };

  posts.push(newPost);
  localStorage.setItem("posts", JSON.stringify(posts));
  document.getElementById("postContent").value = ""; // Clear input
  displayPosts();
}

// Like Post Function
function likePost(postIndex) {
  posts[postIndex].likes++;
  localStorage.setItem("posts", JSON.stringify(posts));
  displayPosts();
}

// Dislike Post Function
function dislikePost(postIndex) {
  posts[postIndex].dislikes++;
  localStorage.setItem("posts", JSON.stringify(posts));
  displayPosts();
}

// Share Post Function
function sharePost(postIndex) {
  alert("Post shared: " + posts[postIndex].content);
}

// Add Comment Function
function addComment(event, postIndex) {
  if (event.key === "Enter") {
    const commentInput = document.getElementById(`comment-input-${postIndex}`);
    const comment = commentInput.value;

    if (comment.trim()) {
      posts[postIndex].comments.push(comment);
      localStorage.setItem("posts", JSON.stringify(posts));
      commentInput.value = ""; // Clear input
      displayPosts();
    }
  }
}

// Follow User Function
function followUser(username) {
  alert("You are now following " + username);
}

// Search Posts Function
function searchPosts() {
  const searchTerm = document.getElementById('search').value.toLowerCase();
  const filteredPosts = posts.filter(post => post.content.toLowerCase().includes(searchTerm));
  displayFilteredPosts(filteredPosts);
}

// Display Filtered Posts
function displayFilteredPosts(filteredPosts) {
  const postsContainer = document.getElementById("posts");
  postsContainer.innerHTML = '';

  filteredPosts.forEach((post, index) => {
    const postDiv = document.createElement("div");
    postDiv.classList.add("postItem");

    postDiv.innerHTML = `
      <p><strong>${post.user}</strong>: ${post.content}</p>
      <button onclick="likePost(${index})">Like (${post.likes})</button>
      <button onclick="dislikePost(${index})">Dislike (${post.dislikes})</button>
      <button onclick="sharePost(${index})">Share</button>
      <button onclick="followUser('${post.user}')">Follow ${post.user}</button>
      <div class="comments">
        <input type="text" id="comment-input-${index}" placeholder="Write a comment..." onkeypress="addComment(event, ${index})">
        <div id="comments-${index}">${post.comments.join('<br>')}</div>
      </div>
    `;
    postsContainer.appendChild(postDiv);
  });
}

</script>

</body>
</html>