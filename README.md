# aws-sso-configuration-steps

## Prerequisites
- Root Access to AWS for AWS Organizations Set up
- Setup AWS Organizations
- AWS Organizations > Settings > AWS Single Sign-On > Enabled
- Super Admin privileges access to your company’s G Suite account.

## I. AWS SSO endpoint configuration (Optional)
- Go to AWS SSO console and enable AWS SSO
- Once AWS SSO is enabled, go to AWS SSO Settings page, under User portal, and choose Change. You can then provide your company’s name in order to get an easy portal URL: https://{company}.awsapps.com/start. You can only setup this subdomain once, no further changes are allowed, choose wisely.

## II. G Suite SAML App initialization and metadata recovery
- For this step, head over to https://admin.google.com, select Apps and choose SAML apps
- Add a new app by clicking the + button on the bottom right corner
- Select Set up my own custom app from the list. Do not select Amazon Web Services from the list. This SAML app pre-configuration is make for AWS single account SSO via IAM federated roles. This is not what we are setting up here.
- Download the IDP metadata to your computer as metadata.xml by clicking the DOWNLOAD link from the option 2 part of the popup.
- Click Next
- Keep this browser tab open, we will need to complete the application setup process in step IV.

## III. AWS SSO external IdP configuration with G Suite metadata and AWS metadata recovery
- Open a new browser tab on AWS SSO console.
- Go to the Settings page, under Identity source, and click Change on the Identity source line.
- Choose External identity provider within the list of available identity providers.
- Under Service provider metadata, click the Show individual metadata values button. Take a note of AWS SSO Sing-in URL, ACS URL and issuer URL. There are unfortunately no metadata upload feature available in Google admin for SAML application setup.
- Upload the metadata.xml file you downloaded earlier from Google admin page in the IdP SAML metadata field.
- Choose Next: Review.
- Once you have read the disclaimer and are ready to proceed, type CONFIRM.
- Choose Finish.

## IV. G Suite SAML App configuration with AWS metadata
- Return to Google Admin tab of your browser
- In the Service Provider details window, add Amazon Web Services as an application name and description.
- Fill in AWS SSO ACS URL value from AWS in the ACS URL field.
- Fill in AWS SSO issuer URL value from AWS in the Entity ID field.
- Fill in AWS SSO Sing-in URL in the Start URL field.
- Leave the Signed Response unchecked.
- Leave the default Name ID as the primary email. This will be our unique identifier on AWS side.
- Click Next.
- Do not modify any Mapping in the Attribute Mapping section.
- Click Finish.

## V. AWS SSO users provisioning
- AWS SSO uses SCIM (System for Cross-domain Identity Management) to do automatic provisioning of users based on IdP information. Unfortunately, no SCIM option is exposed for G Suite IdP. This automatic provisioning method is mainly used for Azure Active Directory federated AWS SSO configuration.
- Therefore, when setting up AWS SSO with G Suite accounts, you need to manually add user in AWS SSO interface. The management of user permission is also made from AWS SSO interface.
- On your existing AWS SSO console browser tab, go to the Settings page, under Identity source, and check that Provisioning line shows Manual.
- Go to the Users page, and click Add user
- Fill in user G Suite email address in both Username and Email address fields. Username field must contain the user email address as we provisioned the G Suite SAML App to use the email as Name ID SAML field. In order to test this AWS SSO setup, I suggest your start by creating your own user.
- You can fill in remaining required field as per your likings.
- Click Next: Groups.
- Click Finish.

## VI. AWS SSO accounts and users management
- Go to AWS accounts page, on Permission sets tab, click Create permission set
- For the purpose of this setup, we will create a predefined Permission set with Administrator privileges. Choose Use an existing job function policy and select AdministratorAccess. The permission set system in place for AWS SSO is very similar to AWS IAM Policy management. The permission set created in AWS SSO are not available as IAM policies and vice-versa.
- Click Create.
- Go to AWS accounts page, on AWS organization tab
- Select one or more accounts within the list of AWS accounts within you organization that you would like the newly created user to have AdministratorAccess to.
- Click Assign user.
- Select your newly created user in the user list and click Next: Permission sets.
- Select your newly created Administrator permission set in the list and click Finish.

## Testing
- You can now go to the URL you configured in the first step (https://{company}.awsapps.com/start) , authenticate via google with G Suite account and be redirected to AWS SSO user portal. In this portal, listed under AWS Account, you can see each AWS Account of your AWS Organization for which your user has at least one permission set. And for each AWS Account listed, you can see a list of available permission set you can use both within the web management console and the command line.

## References
- https://medium.com/serverless-transformation/setup-aws-organizations-with-google-suite-saml-sso-7e676f5ed3e1
- G Suite SAML App setup: https://support.google.com/a/answer/6087519?hl=en
- AWS SSO external IdP setup: https://docs.aws.amazon.com/singlesignon/latest/userguide/manage-your-identity-source-idp.html
