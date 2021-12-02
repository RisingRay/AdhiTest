# AdhiTest

package com.ivasystems.nexa;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.TooltipCompat;

import android.app.Activity;
import android.app.ProgressDialog;
import android.content.Intent;
import android.os.AsyncTask;
import android.os.Bundle;
import android.text.TextUtils;
import android.util.Log;
import android.util.Patterns;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.ProgressBar;
import android.widget.Spinner;
import android.widget.TextView;

import com.loopj.android.http.RequestParams;

import java.util.ArrayList;
import java.util.regex.Pattern;

public class SignUp_AgriOfficer extends AppCompatActivity {
    ImageButton back;
    Button signUp;
    Intent intent;
    TextView signIn;
    String url;
    String designation;
    EditText staff_nameField, mobileNumberField, emailAddressField, passwordField, cnf_passwordField;
    String staff_name, mobileNumber, emailAddress, password, cnf_password;
    String[] stringArray;

    String jobTitle, district, block, agri_Office, userRole;

    String status;
    GeneralFunctionalities generalFunct;
    ProgressBar progressBar2;
    ProgressDialog mProgressDialog;
    DatabaseHelper famaDB;
    Spinner spinner1, spinner2, spinner3, spinner4;
    private AsyncTask progressDialogTask;
    private Activity mActivity;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sign_up_agri_officer);


        mActivity = SignUp_AgriOfficer.this;
        spinner1 = (Spinner) findViewById(R.id.jobTitle);
        TooltipCompat.setTooltipText(spinner1, "Enter Job Title");
        spinner2 = (Spinner) findViewById(R.id.district);
        TooltipCompat.setTooltipText(spinner2, "Enter District");
        spinner3 = (Spinner) findViewById(R.id.block);
        TooltipCompat.setTooltipText(spinner3, "Enter Block");
        spinner4 = (Spinner) findViewById(R.id.agri_office);
        TooltipCompat.setTooltipText(spinner4, "Enter Agri Office");
        initializeAccessorValues();
        // setSpinnerDropdownValues();


        signUp.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (generalFunct.isNetworkAvailable()) {
                    getSignUpFormValues();

                    validateFormFieldValues();
                } else {
                    generalFunct.displayToastMessage("Internet Connectivity Not available");
                }


            }
        });

        back.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                navigateToPage(SignUp_Home.class);
            }
        });
        signIn.setOnClickListener(new View.OnClickListener() {
                                      @Override
                                      public void onClick(View v) {
                                          navigateToPage(FamaLoginPage.class);
                                      }
                                  }
        );
    }

    public void createNewAccountUsingSignUpData() {

        progressDialogTask = new AsyncSendSignUpDetailsTask().execute(4000);

    }

    public void navigateToPage(Class className) {

        intent = new Intent(SignUp_AgriOfficer.this, className);
        startActivity(intent);
    }

    public void initializeAccessorValues() {
        back = findViewById(R.id.btn_backButton);
        signUp = findViewById(R.id.btn_signup);
        famaDB = new DatabaseHelper(this);
        generalFunct = new GeneralFunctionalities(getApplicationContext());
        progressBar2 = (ProgressBar) findViewById(R.id.progressBar2);
        url = GeneralFunctionalities.FAMA_SignUp_URL;
        signIn = findViewById(R.id.accnt2);
        stringArray = new String[0];
        setmProgressDialog();
        setSpinnerDropdownValues();

    }

    public void getSignUpFormValues() {
        jobTitle = ((Spinner) findViewById(R.id.jobTitle)).getSelectedItem().toString();
        district = ((Spinner) findViewById(R.id.district)).getSelectedItem().toString();
        block = ((Spinner) findViewById(R.id.block)).getSelectedItem().toString();
        agri_Office = ((Spinner) findViewById(R.id.agri_office)).getSelectedItem().toString();
        staff_nameField = findViewById(R.id.staff_name);
        staff_name = staff_nameField.getText().toString();
        mobileNumberField = findViewById(R.id.mobilenumber);
        mobileNumber = mobileNumberField.getText().toString();
        emailAddressField = findViewById(R.id.emailAddress);
        emailAddress = emailAddressField.getText().toString();
        passwordField = findViewById(R.id.password);
        password = passwordField.getText().toString();
        cnf_passwordField = findViewById(R.id.cnf_password);
        cnf_password = cnf_passwordField.getText().toString();
    }

    public void validateFormFieldValues() {
        // famaDB.getuserRolesId(designation);
        if (isDetailsComplete()) {
            createNewAccountUsingSignUpData();
        }
    }

    boolean isEmail(EditText text) {
        boolean result = false;

        CharSequence emailAddress1 = text.getText().toString();
        if (Patterns.EMAIL_ADDRESS.matcher(emailAddress1).matches()) {
            result = true;
        } else {
            text.setError("Invalid E-mail");
        }
        return result;
    }

    //Corrected by Adheesh on 13-5-2021/// for Mobile number validation
    boolean isValidMobile(EditText text) {
        boolean result = false;
        CharSequence mobileNumber1 = text.getText().toString();
        String regex = "^[0-9]{10}$";
        if ((Pattern.matches(regex, mobileNumber1)) && (mobileNumber1.length() == 10)) {
            result = true;
        } else {
            text.setError("Invalid Mobile");
        }
        return result;
    }

    boolean passwordIsMatching() {


        if (password.trim().matches(cnf_password.toString().trim())) {

            return true;

        } else {
            return false;
        }
    }

    boolean isEmpty(EditText text) {
        CharSequence str = text.getText().toString();

        return TextUtils.isEmpty(str);

    }

    boolean isIncorrectOption(String selectedValue) {

        return selectedValue.contains("Select");

    }

    //    boolean isPhone(EditText text){
//        CharSequence mobileNumber=text.getText().toString();
//        return(!TextUtils.isEmpty(mobileNumber)&& Patterns.PHONE.matcher(mobileNumber).matches());
//    }


    //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
    public void setSpinnerDropdownValues() {
        setJobTitleDropdownValues();
        setDistrictDropdownValues();
        setBlockDropdownValues();
        setAgri_OfficeDropdownValues();
    }


    public void setJobTitleDropdownValues() {
        ArrayList jobTitleList;
        jobTitleList = famaDB.fetchUserTitlesFromTable("5");
        jobTitleList.add(0, "Select Job Title..");
        ArrayAdapter<String> dataAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, jobTitleList);
        spinner1.setAdapter(dataAdapter);
    }

    public void setDistrictDropdownValues() {
        ArrayList districtList;
        districtList = famaDB.fetchDistrictFromTable();
        Log.i("Inside district", districtList.toString());
        districtList.add(0, "Select District..");
        ArrayAdapter<String> dataAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, districtList);
        spinner2.setAdapter(dataAdapter);
    }

    public void setBlockDropdownValues() {
        ArrayList blockList = new ArrayList<String>();
        blockList = famaDB.fetchBlockFromTable();
        blockList.add(0, "Select Block..");
        ArrayAdapter<String> dataAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, blockList);
        spinner3.setAdapter(dataAdapter);
    }

    public void setAgri_OfficeDropdownValues() {
        ArrayList agriOfficeList;
        agriOfficeList = famaDB.fetchAgriOfficeFromTable();
        agriOfficeList.add(0, "Select Agiculture Office..");
        ArrayAdapter<String> dataAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, agriOfficeList);
        spinner4.setAdapter(dataAdapter);
    }


    //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


    //Corrected by Adheesh on 13-5-2021/// for all fields validation

    boolean isDetailsComplete() {
        boolean isDetailsComplete = true;
        if (isEmpty(staff_nameField) || isEmpty(passwordField) || isEmpty(cnf_passwordField) || isEmpty(emailAddressField)) {
            isDetailsComplete = false;
            Log.i("Inside isempty", "false");
            generalFunct.displayToastMessage("Field Values Incomplete");
            return isDetailsComplete;

        }

        if (isIncorrectOption(jobTitle) || isIncorrectOption(district) || isIncorrectOption(block) || isIncorrectOption(agri_Office)) {
            isDetailsComplete = false;
            Log.i("Inside drop check", "false");
            generalFunct.displayToastMessage("Selected Option Invalid");
            return isDetailsComplete;

        }


        if (!isValidMobile(mobileNumberField) || !isEmail(emailAddressField)) {
            isDetailsComplete = false;
            Log.i("Inside mobile", "false");
            generalFunct.displayToastMessage("Incorrect Mobile num/Email");
            return isDetailsComplete;
        }

        if (!passwordIsMatching()) {
            Log.i("Inside pwd", "false");
            isDetailsComplete = false;
            generalFunct.displayToastMessage("Password and Confirm password should match");
        }
        return isDetailsComplete;

    }

    public RequestParams getRequestParams() {
        RequestParams params = new RequestParams();
        params.put("roleId", "2");
        params.put("name", staff_name);
        params.put("mobilenumber", mobileNumber);
        params.put("email", emailAddress);


        params.put("userInstallationKey", getFAMAUserInstallationKey());
        params.put("userNotificationKey", getFAMAUserNotificationKey());

        params.put("userTitlesId", famaDB.getUserTitlesId(jobTitle));
        //params.put("userTitlesId",  "1");
        params.put("userprofile_departmentid", "-100");

        // params.put("blockId", famaDB.getBlockId(block));
        // params.put("officeId", famaDB.getAgriOfficeId(agri_Office));
        params.put("districtId", famaDB.getDistrictId(district));

        params.put("blockId", "2");
        params.put("officeId", "2");
        params.put("userMobmodel", generalFunct.getDeviceName());
        params.put("password", password);

        Log.i("The Parameters are ", params.toString());
        return params;
    }


    public void setmProgressDialog() {
        mProgressDialog = new ProgressDialog(mActivity);
        mProgressDialog.setProgressStyle(ProgressDialog.STYLE_SPINNER);
        mProgressDialog.setIndeterminate(true);
        mProgressDialog.setMessage("Please wait...Connecting to Server");
        // Progress dialog horizontal style


    }


    public String getFAMAUserInstallationKey() {
        return GeneralFunctionalities.getFAMAUserInstallationKey(getApplicationContext());
    }

    public String getFAMAUserNotificationKey() {
        return GeneralFunctionalities.getFAMAUserNotificationKey(getApplicationContext());
    }

    ///////Modified by Sruthi/////On 25-06-2021////////////////
    public class AsyncSendSignUpDetailsTask extends AsyncTask<Integer, Void, String> {


        @Override
        protected void onPreExecute() {
            Log.i("Inside preexcute", mProgressDialog.toString());
            mProgressDialog.show();


        }


        protected String doInBackground(Integer... milliSec) {

            // TODO Auto-generated method stub
            String famaResponseString = "";


            try {
                Thread.currentThread();
                Thread.sleep(milliSec[0]);
                Log.i("Inside do it", "Do in background");
                famaResponseString = generalFunct.postFAMARequests(url, getRequestParams(), "Sign Up");


            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }


            return famaResponseString;
        }


        @Override
        protected void onPostExecute(String responseString) {

            //Log.i("Sign Up Response", responseString);
            status = generalFunct.responseStatus;
            //Log.i("Response Status is", status);
            if (status.equals("true")) {
                generalFunct.validateFAMASignUp(responseString);
                generalFunct.displayToastMessage(generalFunct.responseMessage);
                navigateToPage(FamaLoginPage.class);
            } else {
                if (status.equals("false")) {
                    generalFunct.validateFAMASignUp(responseString);
                    generalFunct.displayToastMessage(generalFunct.responseMessage);
                } else
                    generalFunct.displayToastMessage(generalFunct.responseMessage);
            }
            mProgressDialog.dismiss();

        }

    }
}


