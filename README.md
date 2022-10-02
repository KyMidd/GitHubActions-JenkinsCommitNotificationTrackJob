# PR Validate, _Any Job Run

This Action is intended for PRs in GitHub that use the xxxxx_Any job on Jenkins for validations. It sends a Jenkins Commit Notification to Jenkins, then uses that information to identify the xxx_Any job name. It finds all jobs with that names and checks the git commit SHA that drove them to be created. When it finds a match, it latches onto that job and starts tracking it. When that job finishes, it translates the exit message from Jenkins into an exit code and surfaces it to GitHub. 

This Action is intended to test code from GitHub and block PRs from being merged if there is a failure. 

Lots more information on Medium at: https://medium.com/@kymidd/lets-do-devops-github-to-jenkins-custom-integration-using-actions-bash-curl-for-api-hacking-e1089bc29a56

# Example call of this action

```
name: PR Validate _Any Job

# Run this job on pull request with any action, which includes opening and any commit
on: [pull_request]

jobs:
  jenkins_pr_validate_any:
    runs-on: [self-hosted, YourInternalRunners]
    
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Identify PR commit
        run: |
          # Interestingly, GITHUB Actions are served a git log with an extra commit
          # Genuinely no idea why, so we grab the appropriate commit from the log
          commit_sha=$(git log | grep commit | sed -n 2p | cut -d " " -f 2)
          
          # Write commit sha to GITHUB_ENV so downstream tasks can use it
          echo "commit_sha=$commit_sha" | tee -a $GITHUB_ENV
      
      - name: Jenkins Any PR Validate
        id: jenkins_any_pr_validate
        uses: KyMidd/GitHubActions-JenkinsCommitNotificationTrackJob@v1
        with:
          jenkins-username: "${{ secrets.JENKINS_USERNAME }}"
          jenkins-api-token: "${{ secrets.JENKINS_API_TOKEN }}"
          github_repository: $GITHUB_REPOSITORY
          commit_sha: ${{ env.commit_sha }}
          branch: $GITHUB_HEAD_REF
```