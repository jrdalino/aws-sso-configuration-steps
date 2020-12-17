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
- Reference: https://support.google.com/a/answer/6087519
- Step 1: Go to https://admin.google.com/
- Step 2: Click on Apps > SAML Apps > Add App > Add Custom SAML app 
- Step 3: App Details > App Name: AWS SSO
- Step 4: Google Indentity Provider Details > Click On Option 1: Download IdP Metadata (GoogleIDPMetadata.xml) > Press Continue
- Step 5: Service Provider Details > Add the following details
```
> ACS URL (actual):     https://ap-southeast-1.signin.aws.amazon.com/platform/saml/acs/55b77c17-630f-4cb2-ade9-8e398ee78bee
> Entity Id (actual):   https://ap-southeast-1.signin.aws.amazon.com/platform/saml/d-96671afd19
> Start URL (optional): https://wknc.awsapps.com/start
> Signed Response:      Leave Unchecked
> Named Id format:      Leave UNSPECIFIED
> Name Id:              Leave Basic Information > Primary email
```
- Step 6: Attribute mapping > Add Mapping and Click on "Finish"
- Step 7: User access > Click on "ON for everyone" > Click Save

## III. AWS SSO external IdP configuration with G Suite metadata and AWS metadata recovery
- Open a new browser tab on AWS SSO console.
- Go to the Settings page, under Identity source, and click Change on the Identity source line.
- Choose External identity provider within the list of available identity providers.
- Under Service provider metadata, click the Show individual metadata values button. Take a note of AWS SSO Sign-in URL, ACS URL and issuer URL. There are unfortunately no metadata upload feature available in Google admin for SAML application setup.
- Upload the metadata.xml file you downloaded earlier from Google admin page in the IdP SAML metadata field.
- Choose Next: Review.
- Once you have read the disclaimer and are ready to proceed, type CONFIRM.
- Choose Finish.
- AWS SSO-integrated applications > Enable in member accounts

## IV. AWS SSO users provisioning
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
