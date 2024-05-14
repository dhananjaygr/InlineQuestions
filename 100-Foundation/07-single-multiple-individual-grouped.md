## Analyze scan results in Wiz

1. In the Wiz portal, navigate to **Reports > Version Control Scans**.

1. Find your triggered scan. On the `Repository` column, look for your <github_user>/<repo_name>. Or you can use the *Repository* filter.

    <p align="left">
      <img width="680" height="176" img src="pics/repo_scans.png"/>
      </p> 

1. Open the details of your scan, lets look into some of the information provided.

    - **Event Properties** shows basic details of the scan, including the overall severity and trigger type (Pull Request or Scheduled)
    - **CI/CD Scan** shows the scan result and the number of findings per severity.
    - **Properties** shows the scan duration and time, and most importantly, what scan policies are enabled at the connector level.

        <p align="left">
          <img width="750" height="573" img src="pics/cicd_scan1.png"/>
          </p> 

1. Now lets look at the details of the **Pull Request Properties** section:

    - This section shows who committed the code (**Author**), the branch name (**Title**), the base and committed hashes (**Base Commit Ref** and **Head Commit Ref**), number of commits in the PR, number of files added, and the PR url.

        <p align="left">
          <img width="750" height="161" img src="pics/cicd_scan2.png"/>
          </p> 


1. The **Contains Configuration Findings** and **Secret Instances** sections show the IaC misconfiguration and secret findings respectively. 

    - We will address the problems in the next step, but you can see Wiz found 4 configuration findings, with one being of High severity.

       <p align="left">
         <img width="750" height="323" img src="pics/cicd_scan3.png"/>
         </p> 


## Address code commit issues

1. Back at your Github's PR page, within the check step, find the `Wiz IaC Scanner` line and click on **Details**.

   <p align="left">
      <img width="650" height="274" img src="pics/check_fail.png"/>
      </p> 

1. Here you will see the High severity finding within the `Dockerfile` file. 

    `The 'Dockerfile' should contain the 'USER' instruction`

    <p align="left">
      <img width="650" height="328" img src="pics/iac_scan_detail.png"/>
      </p> 

1. Find the link to `Wiz Secret Scanner` and click on it.

    <p align="left">
      <img width="350" height="167" img src="pics/wiz_secret.png"/>
      </p> 

1. Here you will see the Secret finding, where we committed an AWS Access/Secret Key pair within our code.

    <p align="left">
      <img width="650" height="306" img src="pics/secret_scan_detail.png"/>
      </p> 

    In a production scenario, you would not want to add Access/Secret Keys inside your code. This is extremely dangerous, and can lead to compromise and priviledge escalation. There are alternatives like using an AWS role attached to the instances running the containers/application. 
    
    As part of this lab, we will fix the Dockerfile problem, but leave the secret. We want to show that it's possible to bypass the PR checks (although not recommended)

1. From the PR's Checks page, navigate to **< > Code**.

1. Change your branch to `task1`.

    <p align="left">
      <img width="400" height="405" img src="pics/switch_branch.png"/>
      </p> 

1. Open the `Dockerfile` file.

1. Update the file to look like the one below. You can copy and paste if you wish.

      <div style="margin-right: 150px;">

      ```
      FROM ubuntu:16.04
      RUN mkdir -p /home/se/.aws/ \
          groupadd -r appuser && useradd -r -g appuser appuser \
          chown -R appuser:appuser /var/www /var/log/apache2 /etc/apache2
      COPY awssecret.json /home/se/.aws/credentials
      RUN apt-get update && \
          apt-get install -y apache2=2.4.18-2ubuntu3.17
      USER appuser
      EXPOSE 80
      HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
        CMD curl -f http://localhost/ || exit 1
      CMD ["apache2ctl", "-D", "FOREGROUND"]
      ```
      </div>

1. Click on **Commit changes...**. Select **Commit directly to the *task1* branch**. Click on **Commit changes**.

1. Click on **Pull Requests**, then select `Task1`.

1. In the conversation tab, scroll down to the test step. You should see that now the `Wiz IaC Scanner` is in *Neutral* state.

    <p align="left">
      <img width="650" height="271" img src="pics/check_neutral.png"/>
      </p> 

1. Click on details, you will see that there are still some checks with severity 'None' (informational). Those are non-blocking and we are OK to continue.

    <p align="left">
      <img width="650" height="349" img src="pics/neutral_detail.png"/>
      </p> 

1. Select `Wiz Secret Scanner`, you will still see the check failed, but as discussed earlier, we will ignore the error in this exercise.

1. Go back to the **Conversation** tab of your PR and scroll down to the check step.

1. Click on **Merge without waiting for requirements to be met (bypass branch protections)**, then click on **Merge pull request**.

    <p align="left">
      <img width="650" height="272" img src="pics/merge_pr.png"/>
      </p> 

1. Click on **Confirm merge**.

1. Congratulations, your Pull Request is merged! The code is now in the `main` branch, which in our case means "production". 

    <p align="left">
      <img width="650" height="72" img src="pics/pr_merged.png"/>
      </p> 

1. Click on **Delete branch** to delete your merged `task1` branch.

1. Back into the Wiz portal, navigate to **Reports > Version Control Scans**, and find your latest scan. *Tip* use the Repository filter. 

1. Open your latest scan, scroll down to the **Contains Configuration Findings** section. Here you will see only the two informational findings which matches the GitHub scan you've seen earlier.

    <p align="left">
      <img width="650" height="123" img src="pics/cc_findings.png"/>
      </p> 

</div>
