# Deploying a Flask API

This is the project starter repo for the fourth course in the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004): Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Initial setup
1. Fork this project to your Github account.
2. Locally clone your forked version to begin working on the project.

## Dependencies

- Docker Engine
    - Installation instructions for all OSes can be found [here](https://docs.docker.com/install/).
    - For Mac users, if you have no previous Docker Toolbox installation, you can install Docker Desktop for Mac. If you already have a Docker Toolbox installation, please read [this](https://docs.docker.com/docker-for-mac/docker-toolbox/) before installing.
 - AWS Account
     - You can create an AWS account by signing up [here](https://aws.amazon.com/#).
     
## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson [here](https://classroom.udacity.com/nanodegrees/nd004/parts/1d842ebf-5b10-4749-9e5e-ef28fe98f173/modules/ac13842f-c841-4c1a-b284-b47899f4613d/lessons/becb2dac-c108-4143-8f6c-11b30413e28d/concepts/092cdb35-28f7-4145-b6e6-6278b8dd7527).


```
export JWT_SECRET='myjwtsecret'

export LOG_LEVEL=DEBUG
```



```
export TOKEN=`curl -d '{"email":"marvin.abisrror@gmail.com", "password": "kitty"}' -H "Content-Type: application/json" -X POST localhost:8080/auth | jq -r '.token'`

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   252  100   194  100    58  14923   4461 --:--:-- --:--:-- --:--:-- 19384
(cntrepo) mabisrror@marvins-MacBook-Pro docker % echo $TOKEN
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MDI2NTQwNDgsIm5iZiI6MTYwMTQ0NDQ0OCwiZW1haWwiOiJtYXJ2aW4uYWJpc3Jyb3JAZ21haWwuY29tIn0.TnTs5tu7AG64a6h3hyB-Urrg1VSAedABD-jt8hFbBdQ

```


```
curl --request GET 'http://127.0.0.1:8080/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    87  100    87    0     0  12428      0 --:--:-- --:--:-- --:--:-- 10875
{
  "email": "marvin.abisrror@gmail.com",
  "exp": 1602654048,
  "nbf": 1601444448
}
```

--------------------


```
export TOKEN=`curl -d '{"email": "marvin.abisrror@gmail.com", "password": "kitty"}' -H "Content-Type: application/json" -X POST localhost:80/auth  | jq -r '.token'`

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   248  100   189  100    59   7269   2269 --:--:-- --:--:-- --:--:--  9538

$TOKEN
```



```
curl --request GET 'http://127.0.0.1:80/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    72  100    72    0     0   6545      0 --:--:-- --:--:-- --:--:--  6545
{
  "email": "marvin.abisrror@gmail.com",
  "exp": 1602739983,
  "nbf": 1601530383
}
```

----------------------
----------------------
Once you submit your project, and receive the reviews, you can consider deleting the variable from parameter-store using:

aws ssm delete-parameter --name JWT_SECRET



