# Project Implementation

## DEV part - Running the project locally
- Create AWS Account
- Create an EC2 instance
  - Region - London (eu-west-2)
  - Name - netflix-jenkins
  - AMI - Ubuntu
  - Instance type - t2.large
  - Create new key pair (if you don't have one)
    - name: netflix_key_pair, RSA,.pem
  - Network settings - Allow SSH, HTTP, HTTPS traffic
  - Configure storage - Size: 25 GiB
- Allocate Elastic IP to your Instance
- Add rules to the security group
  - 8081, Anywhere IPv4, app port
  - 8080, Anywhere IPv4, jenkins
  - 9000, Anywhere IPv4, sonarqube
- Connect to the Instance using EC2 Instance Connect
- sudo apt update -y
- Clone the repository on the EC2 instance
- Navigate to the repository DevSecOps-Project (you'll see the Dockerfile)
 

- In the browser, go to 'TMDB' website and create an account
- Go to your profile, create and copy the API Key


- Install Docker on your EC2 instance - check using 'docker version' command
- docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix . - check using 'docker images', netflix image should be created
- docker run -d -p 8081:80 netflix
- Copy the PublicIP of the instance, in the browser search <PublicIP>:8081 - will only open if port 8081 is open in the security group
- The Netflix clone website should be displayed successfully  




