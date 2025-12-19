# hello-world-npm
Hello World. I'm curious how does npm publishing with CI/CD works 


- Due to [restriction](https://docs.npmjs.com/trusted-publishers#supported-cicd-providers) in `Note: Each package can only have one trusted publisher configured at a time.` all publishing must be setup in the same yml action file 
- Package json version is updated automatically on selected target branch, don't do it manually 
- When latest is selected it's tag latest in npm and github release 
- When `pre-release` is selected it's should be deprecated automatically but it's impossible with OIDC so far on npn and tagged as `pre-release` 