# awsCognitoReceiveData
Receive Data From Aws Cognito

Download API in the above file (AppHelper, ItemToDisplay)

Enable AWS Mobile Hub Login:
```java
//Initialize AWS Sign in
AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
     @Override
     public void onComplete(AWSStartupResult awsStartupResult) {
         showSignIn();
     }
}).execute();
//END Initialize AWS Sign in
```

After login to mobile hub, Initialize AWS Cognito:
```java
IdentityManager.getDefaultIdentityManager().addSignInStateChangeListener(new SignInStateChangeListener() {
    @Override
    public void onUserSignedIn() {
       Log.d("AWSCheck", "User Sign In");
       //Initialize Cognito Api
       AppHelper.init(getApplicationContext());
    }

    @Override
    public void onUserSignedOut(){
        Log.d("AWSCheck", "User Sign Out");
        showSignIn();
    }
});

private void showSignIn(){
    Log.d("AWS", "showSignIn");
    AuthUIConfiguration configuration = new AuthUIConfiguration.Builder()
            .userPools(true)
            .logoResId(R.drawable.application_logo)
            .backgroundColor(R.color.application_background_color)
            .isBackgroundColorFullScreen(true)
            .build();

    SignInUI signin = (SignInUI) AWSMobileClient.getInstance().getClient(LoginActivity.this, SignInUI.class);
    signin.login(LoginActivity.this, MainActivity.class).authUIConfiguration(configuration).execute();
}
```

Get User Name:
```java
private void showWelcomeMessage(){
        CognitoUser user = AppHelper.getPool().getCurrentUser();
        String username = user.getUserId();
        if(username != null) {
            AppHelper.setUser(username);
            Snackbar.make(findViewById(R.id.main_activity), "Welcome back, " + username, Snackbar.LENGTH_SHORT).show();
        }
}
```

Get User Details
```java
//Define CognitoUser
CognitoUser user = AppHelper.getPool().getCurrentUser();

//Sync and download user details
GetDetailsHandler getDetailsHandler = new GetDetailsHandler() {
    @Override
    public void onSuccess(CognitoUserDetails cognitoUserDetails) {
        //Store details in App Handler
        AppHelper.setUserDetails(cognitoUserDetails);
        Map<String, String> user = cognitoUserDetails.getAttributes().getAttributes();
        for(Map.Entry<String, String> entry : user.entrySet()){
            profileKey.add(entry.getKey());
            profileValue.add(entry.getValue());
            Log.d("check", profileKey.toString());
            Log.d("check", profileValue.toString());
        }
    }

    @Override
    public void onFailure(Exception exception) {
        Snackbar.make(getView(), "Fetching Failure", Snackbar.LENGTH_SHORT).show();
    }
};
AppHelper.getPool().getUser(user.getUserId()).getDetailsInBackground(getDetailsHandler);
//END Sync and download user details
```

Reference:
```
1. aws-sdk-android-samples/AmazonCognitoYourUserPoolsDemo/:
https://github.com/awslabs/aws-sdk-android-samples/tree/master/AmazonCognitoYourUserPoolsDemo

2. Getting User Attributes from Cognito User Pool
https://stackoverflow.com/questions/49596835/getting-user-attributes-from-cognito-user-pool
```
