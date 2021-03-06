# Reference documentation for the API.

#### Index

  - [Roles](#roles)
    - [Roles with their access rights](#roles-with-their-access-rights)
  - [Error Reporting](#error-reporting)
  - [Routes](#routes)
    - [Authentication & Authorization](#authentication--authorization)
      - [Login](#login)
      - [Refreshing the JWT token](#refreshing-the-jwt-token)
  - [Health Check](#health-check)
  - [Paginated Results](#paginated-results)
  - [Users](#users)
    - [User Resource](#user-resource)
    - [Creating Users](#creating-users)
      - [Create user profile](#create-user-profile)
      - [Creating user profiles from CSV file:](#creating-user-profiles-from-csv-file)
    - [Getting Users](#getting-users)
      - [Get all user profiles](#get-all-user-profiles)
      - [Get currently logged in user profile](#get-currently-logged-in-user-profile)
      - [Get specific user profile](#get-specific-user-profile)
    - [Updating User](#updating-user)
      - [Update currently logged in user's password](#update-currently-logged-in-users-password)
      - [Update current user's profile](#update-current-users-profile)
      - [Updating user's profile (any user)](#updating-users-profile-any-user)
    - [Deleting Users](#deleting-users)
      - [Delete user profile](#delete-user-profile)
      - [Deleting current user's profile](#deleting-current-users-profile)
    - [Blocked Users](#blocked-users)
  - [Issues](#issues)
    - [Creating Issues](#creating-issues)
      - [Create Issue](#create-issue)
    - [Getting Issues](#getting-issues)
      - [Getting All Issues](#getting-all-issues)
      - [Get all issues](#get-all-issues)
      - [Get all resolved issues](#get-all-resolved-issues)
      - [Get all unresolved issues](#get-all-unresolved-issues)
      - [Get issues by phrase:](#get-issues-by-phrase)
      - [Get issue by ID](#get-issue-by-id)
      - [Get issues by created by currently logged in user](#get-issues-by-created-by-currently-logged-in-user)
      - [Getting comments](#getting-comments)
      - [Getting solutions](#getting-solutions)
    - [Updating issue](#updating-issue)
      - [Toggle issue resolve status](#toggle-issue-resolve-status)
      - [Post a comment on issue](#post-a-comment-on-issue)
      - [Post a solution on issue](#post-a-solution-on-issue)
    - [Deleting issue](#deleting-issue)
      - [Delete an issue](#delete-an-issue)
  - [Stats](#stats)
      - [Get Issues Stats](#get-issues-stats)
      - [Get Authority Stats](#getting-authority-stats)
## Roles

### Roles with their access rights

- **student**: Can view, update & delete own profile. Create, read, comment & upvote related issues, and mark issues created by them as `resolved` or delete them.
- **student_moderator**: Extends all the rights of `student`. Additionally, can mark an issue as `inappropriate` or `resolved`.
- **auth_level_one**: All the rights of `student`. Additionally, can post solutions to issues.
- **auth_level_two**: Extends all the rights of `auth_level_one`. Additionally, can create, read, update & delete any user profile.
- **auth_level_three**: Extends all rights of `auth_level_two`.
  
**[⬆Back to index](#index)**


---

## Error Reporting
_Error reporting format_:
```JSON
statusCode: [401, 403, 500]

{
  "code": "<Error code>",
  "result": "FAILURE",
  "message": "<Error message>",
}
```

- `code` and `message` would be specified in the `String` format.
  
**[⬆Back to index](#index)**

---

## Routes

_All the requests must send the data in_ `application/JSON` _format only_.

### Authentication & Authorization

#### Login

**Method**: **`POST`**
<br>
**URL**: **`api/auth/login`**
<br>

**Required parameters**:

```JSON
{
  "userId": "<userId>",
  "password": "<password>",
}
```
- userId should be exactly 7 characters long, and in `Integer` format.
- password should be at least 6 characters long,and in `String` format.

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "token": "<generated JWT token>",
  "refreshToken": "<generated refresh token token>",
}
```
- The JWT token expires in a predefined amount of time.
- On the expiry of the token, it must be renewed using the refresh token.
- The provided refresh token is persistent. It must be safely stored on the local storage of the client.

**[⬆Back to index](#index)**

#### Refreshing the JWT token

**Method**: **`POST`**
<br>
**URL**: **`api/auth/refresh`**
<br>

**Required parameters**:
```JSON
{
  "refreshToken": "<refresh token provided at login>"
}
```
>**Note:** The request must contain the expired JWT token in the `authorization` header as `'Bearer [the JWT token]`.

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "token": "<generated JWT token>"
}
```

**[⬆Back to index](#index)**

---

## Health Check
> This a public route accessible to all without the use of access token. Use this route to check the status of the API.

**Method**: **`GET`**
<br>
**URL**: **`/api`**
<br>
**Accessible to**: `All`

_Successful Response format_:
```JSON
statusCode : 200

{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Grievance System API —— Read the API documentation here: https://github.com/Zeal-Student-Developers/issue-tracker/blob/master/Documentation.md"
}
```

---

### Paginated Results

The search results returning a `list` of resources are paginated. The page number and search limit should be passed as query parameters in the URL. Both the page number and search limit are optional. If not provided, the page number defaults to `1` and search limit defaults to `5`.
## Users

>**Note:** All requests must contain the valid non-expired JWT token in the `authorization` header as `Bearer [the JWT token].

### User Resource
The following details about the user are returned:
```JSON
{
  "userId": "user's userId",
  "firstName": "user's firstName",
  "lastName": "user's lastName",
  "department": "user's department",
  "role": "user's role",
}
```

### Creating Users

#### Create user profile

**Method**: **`POST`**
<br>
**URL**: **`api/users/add`**
<br>
**Accessible to**: Only to `auth_level_two` and above.
<br>

**Required parameters**:
```JSON
{
  "userId": "<user's userId>",
  "firstName": "<user's first name>",
  "lastName": "<user's last name>",
  "password": "<user's password>",
  "department": "<user's department>",
  "role": "<user's role>",
}
```
- userId must be exactly 7 characters long, in `Integer` format.
- password must be at least 6 characters, in the `String` format.
- department must be at least 2 characters long, in `String` format.
- role should be any one of the specified roles.

_Successful Response format_:
```JSON
statusCode: 201

{
  "code": "CREATED",
  "result": "SUCCESS",
  "message": "User added",
}
```

**[⬆Back to index](#index)**

#### Creating user profiles from CSV file:

 
**Method**: **`POST`**
<br>
**URL**: **`api/users/add/all`**
<br>
**Accessible to**: `All`

**Required parameters**: The csv file must be sent to the API in `multipart/form-data` format, with the fieldname as `users`.
<br>
The CSV file must contain a header and must be in the following format: 
<table>
  <tr>
    <th>ZPRN</th>
    <th>FIRST_NAME</th>
    <th>LAST_NAME</th>
    <th>PASSWORD</th>
    <th>DEPARTMENT</th>
    <th>ROLE</th>
  </tr>
  <tr>
    <td colspan="6" style="text-align:center;font-style:italic">Data</td>
  </tr>
</table>

_Successful Response format_:
<br>
If there were no errors in creating the users, the API would respond with:

```JSON
statusCode : 200

{
  "code": "OK",
  "result": "SUCCESS",
  "message": "User profiles created",
}
```

The API checks the entire data once before creating the user profiles. If any user has inappropriate/insufficient data, the API would respond with:

```JSON
statusCode : 400

{
  "code": "BAD_REQUEST",
  "result": "FAILURE",
  "message": "Users with userId [list] have inappropriate/insufficient data"
}
```
- The `list` would be a comma-seperated list of userId of users profiles having inappropriate/insufficient data, with parantheses.

> **Note:** If any user has inappropriate/insufficient data, no user profiles are created at all. The client must again make a `POST` request to the API with the entire corrected data.

### Getting Users

#### Get all user profiles

**Method**: **`GET`**
<br>
**URL**: **`api/users/all?page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: Only to `auth_level_two` and above.
<br>

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "user": "[List of users]",
}
```

**[⬆Back to index](#index)**

#### Get currently logged in user profile

**Method**: **`GET`**
<br>
**URL**: **`api/users`**
<br>
**Accessible to**: `All`
<br>

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "user": {
    "userId": "<user's userId>",
    "firstName": "<user's first name>",
    "lastName": "<user's last name>",
    "department": "<user's department>",
  }
}
```

**[⬆Back to index](#index)**

#### Get specific user profile

**Method**: **`GET`**
<br>
**URL**: **`api/users/:id`**
<br>
**Accessible to**: Only to `auth_level_two` and above.
<br>

**Required parameters**: No data should be passed in `application/JSON` format, only valid `userID` should be specified in the **URL**.

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "user": {
    "userId": "<user's userId>",
    "firstName": "<user's first name>",
    "lastName": "<user's last name>",
    "department": "<user's department>",
    "role": "<user's role>",
  }
}
```

**[⬆Back to index](#index)**

### Updating User

#### Update currently logged in user's password

**Method**: **`PATCH`**
<br>
**URL**: **`api/users/update/password`**
<br>
**Accessible to**: `All`
<br>

**Required parameters**:
```JSON
{
  "oldPassword": "<user's current password>",
  "newPassword": "<user's updated password>".
}
```
- oldPassword must not be the same as the newPassword.
- New password must contain at least 6 characters, and in `String` format.

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Password updated",
}
```

**[⬆Back to index](#index)**

#### Update current user's profile

**Method**: **`POST`**
<br>
**URL**: **`api/users/update/profile`**
<br>
**Accessible to**: `All`
<br>

**Required parameters**:
```JSON
{
  "firstName":"<updated firstName>",
  "lastName":"<updated lastName>",
  "department":"<updated department>",
}
```
>**Note**: The currently logged in user can only update their firstName, lastName & department. Only those fields must be passed that are updated, rest must be left undefined.

_Successful response format_
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "message": "User updated"
}
```
**[⬆Back to index](#index)**

#### Updating user's profile (any user)

**Method**: **`POST`**
<br>
**URL**: **`api/users/update/profile/:id`**

**Accessible to**: Only to `auth_level_two` and above.
<br>

**Required parameters**:
```JSON
{
  "firstName": "<user's first name>",
  "lastName": "<user's last name>",
  "password": "<user's password>",
  "department": "<user's department>",
  "role": "<user's role>",
}
```
**Note:** Fields that are not updated, must not be defined.

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "message": "User updated",
}
```
**[⬆Back to index](#index)**

### Deleting Users
>**Note**: No document is deleted from the database, only it's visibility to the API is removed.

#### Delete user profile

**Method**: **`DELETE`**
<br>
**URL**: **`api/users/:id`**
<br>
**Accessible to**: `auth_level_two` & above.
<br>

**Required parameters**: No data should be passed in `application/JSON` format, only valid `userID` should be specified in the **URL**.
<br>

_Successful Response format_:
```JSON
statusCode: 200

{
  "code": "OK",
  "result": "SUCCESS",
  "deleted": "<deleted user>",
}
```
**[⬆Back to index](#index)**

#### Deleting current user's profile

**Method**: **`Delete`**
<br>
**URL**: **`api/users/`**
<br>
**Accessible to**: `All`
<br>

**Required parameters**: `None`

_Successful Response format_:
```JSON
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "User deleted",
}
```
**[⬆Back to index](#index)**

### Blocked Users

Users who repeatedly violate the Code of Conduct may be blocked by the system. The current limit set for Code of Conduct violations is set to `5`. Blocked users are restricted from accessing any of the API endpoints. On recieving a request from a blocked user on any endpoint, the API would respond with
```JSON
statusCode : 403

{
  "code": "FORBIDDEN",
  "result": "FAILURE",
  "message": "You have been blocked by the system for repeated violations of the Code of Conduct"
}

```

---
## Issues

### Creating Issues

### Issue Resource
The following details about the issue are returned:
```JSON
{
  "title": "Issue title",
  "description": "Issue description",
  "images": [
    "List of image URLs",
  ],
  "isEdited": "Is issue edited",
  "isResolved": "Is issue resolved",
  "isInappropriate": "Is issue spam",
  "createdOn": "Date issue was created on",
  "_id": "Issue ID",
  "section": "Issue section",
  "department": "Issue department",
  "scope": "Scope of issue",
  "upvotes": "count of upvotes",
  "commentsCount": "count of comments on issue",
  "solutionsCount": "count of solutions on issue",
  "isAuthor": "Is currently logged in user author of issue",
  "hasUpvoted": "Has currently logged in user upvoted the issue",
  "hasReported": "Has currently logged in user reported the issue",
},
```

#### Create Issue
Creating an issue is a two-step process:

**Step I:** Sending images, if any.

**Method**: **`POST`**
<br>
**URL**: **`api/issues/images`**
<br>
**Accessible to**: `All`

**Required parameters**: The image(s) must be sent in `multipart/form-data` format, with the fieldname as `images`.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "files": "[paths to images]",
}
```
> **Note:** The server only accepts image files of type `jpg`, `jpeg`, `png` and `gif` only. If any one of the files sent is not of one of the above types, none of the files are uploaded, and the server responds with an error.

**Step II:** Sending issue details.

**Method**: **`POST`**
<br>
**URL**: **`api/issues/`**
<br>
**Accessible to**: `All`

**Required parameters**:
```JSON
{
  "title": "title of the issue",
  "description": "description of the issue",
  "images":"[paths to images]",
  "section": "section to which the issue belongs",
  "scope": "scope of the issue, [INTITUTE/DEPARTMENT]",
}
```
- `title` should be in `String` format.
- `description` should be in `String` format.
- `images` must be an array of paths to the images returned by the **STEP 1**. If there are no images, an empty array must be sent.
- `section` must be in `String` format.
- `scope` must be one of the two: `DEPARTMENT` or `INSTITUTE`.


_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Issue created",
}
```
**[⬆Back to index](#index)**

### Getting Issues

>**Note:** The routes returning a list of issues depend on the role of logged in user. Users with role of `auth_level_two` & above get the list of all issues irrespective of the department of the issue. Rest all users get list of issues related to their department or those whose scope is `institute`. 

#### Getting All Issues

#### Get all issues

> Returns a list of all resolved & unresolved issues

**Method**: **`GET`**
<br>
**URL**: **`api/issues/all?page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "issues":"[List of Issues]",
  },
}
```
**[⬆Back to index](#index)**

#### Get all resolved issues
> Returns a list of all unresolved issues only.

**Method**: **`GET`**
<br>
**URL**: **`api/issues/all/resolved?page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "issues":"[List of Issues]",
  },
}
```
**[⬆Back to index](#index)**

#### Get all unresolved issues
> Returns a list of all unresolved issues only.

**Method**: **`GET`**
<br>
**URL**: **`api/issues/all/unresolved?page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "issues":"[List of Issues]",
  },
}
```
**[⬆Back to index](#index)**

#### Get issues by phrase:
**Method**: **`GET`**
<br>
**URL**: **`api/issues/phrase?phrase=[search_phrase]&page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `phrase` as a query parameter in the URL itself

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "issues":"[List of Issues]",
  },
}
```

#### Get issue by ID
**Method**: **`GET`**
<br>
**URL**: **`api/issues/:id`**
<br>
**Accessible to**: `All`

**Required parameters**: `id` of the issue in the request URL itself.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "issue": "[issue]"
}
```

#### Get issues by created by currently logged in user 
**Method**: **`GET`**
<br>
**URL**: **`api/issues/own?page=[page_number]&limit=[search_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "issues":"[List of Issues]",
  },
}
```
**[⬆Back to index](#index)**

#### Getting comments
**Method**: **`GET`**
<br>
**URL**: **`api/issues/:id/comments?page=[page_number]&limit=[page_limit]`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`
 
_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "comments": [
      {
        "comment": "Comment text",
        "postedOn": "Date comment was posted on",
      }
    ],
  },
}
```

#### Getting solutions
**Method**: **`GET`**
<br>
**URL**: **`api/issues/:id/solutions`**
<br>
**Accessible to**: `All`

**Required parameters**: `None`
 
_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "data": {
    "hasNextPage": "[Whether next page is available to fetch]",
    "hasPreviousPage": "[Whether previos page is available to fetch]",
    "solutions": [
      {
        "solution": "Solution text",
        "postedBy": {
          "id": "id of the author",
          "firstName": "First name of the author",
          "lastName": "Last name of the author",
        },
        "postedOn": "Date solution was posted on",
      }
    ],
  },
}
```

### Updating issue

#### Toggle issue resolve status
**Method**: **`PUT`**
<br>
**URL**: **`api/:id/resolve`**
<br>
**Accessible to**: `student` and `student_moderator` only.

**Required parameters**: `id` of the issue in the request URL itself.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Resolve status updated",
}
```
**[⬆Back to index](#index)**
> **Note:** Only the student who created the issue or user with role `student_moderator` can toggle issue resolve status.


#### Post a comment on issue
**Method**: **`PUT`**
<br>
**URL**: **`api/issues/:id/comment`**
<br>
**Accessible to**: `All`

**Required parameters**: `id` of the issue in the request URL itself.
```JSON
{
  "comment":"<comment>"
}
```
- `comment` must be `String` format.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Comment posted",
}
```
**[⬆Back to index](#index)**

#### Post a solution on issue
**Method**: **`PUT`**
<br>
**URL**: **`api/issues/:id/solution`**
<br>
**Accessible to**: `auth_level_two` & above

**Required parameters**: `id` of the issue in the request URL itself.
```JSON
{
  "solution":"<solution>"
}
```
- `solution` must be `String` format.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Solution posted",
}
```
**[⬆Back to index](#index)**

### Deleting issue

#### Delete an issue
**Method**: **`DELETE`**
<br>
**URL**: **`api/issues/:id`**
<br>
**Accessible to**: `All`

**Required parameters**: `id` of the issue in the request URL itself.

_Successful Response format_:
```JSON
statusCode : 200
{
  "code": "OK",
  "result": "SUCCESS",
  "message": "Issue deleted",
}
```
**[⬆Back to index](#index)**

## Stats

#### Get Issues Stats
**Method**: **`GET`**
<br>
**URL**: **`api/stats/issues`**
<br>
**Accessible to**: `auth_level_three`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200

{
  "code": "OK",
  "result": "SUCCESS",
  "stats": {
    "total": "<total issues>",
    "resolved": "<resolved issues>",
    "unresolved": "<unresolved issues>",
    "violatingIssues": "<issues violating code of conduct>",
    "issuesCountByDepartment":"[issues count by department]",
    "violatingUsersCount": "<users posting violating content>",
    "issuesPostedByMonth": "[issues posted by month]", 
    "issuesSolvedByMonth": "[issues solved by month]",
    "topIssues": {
      "byUpvotes": "[List of issues]",
      "byDepartment": "[List of issues]",
    } 
  }
}

```

**[⬆Back to index](#index)**

#### Getting authority stats
**Method**: **`GET`**
<br>
**URL**: **`api/stats/authority`**
<br>
**Accessible to**: `auth_level_three`

**Required parameters**: `None`

_Successful Response format_:
```JSON
statusCode : 200
{
  {
  "code": "OK",
  "result": "SUCCESS",
  "stats": {
    "totalAuthorityUsers": "count of total authority",
    "authorityCountByRole": {
      "auth_level_one": "count",
      "auth_level_two": "count",
      "auth_level_three": "count"
    },
    "authorityCommentsAndSolutions": [
      {
        "name": "Name of authority",
        "commentsCount": "comments count",
        "solutionsCount": "solutions count",
      },
    ],
    "distribution": {"comments count"
      "maxSolutions": {
        "count": "solutions count",
        "name": "Name of authority"
      },
      "minSolutions": {
        "count": "solutions count",
        "name": "Name of authority"
      },
      "maxComments": {
        "count": "comments count",
        "name": "Name of authority"
      },
      "minComments": {
        "count": "comments count",
        "name": "Name of authority"
      }
    }
  }
}
}
```
**[⬆Back to index](#index)**
