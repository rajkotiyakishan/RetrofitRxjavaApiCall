# RetrofitRxjavaApiCall
Calling GET(without passing header) or POST API using Retrofit and RX java and get Response quickly.

First Add Dependency in your build.gradle(Module:app)
 
 // Retrofit
 
 implementation ‘com.squareup.retrofit2:retrofit:2.4.0’
 
 implementation ‘com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4’
 
 implementation ‘com.google.code.gson:gson:2.8.5’
 
 implementation ‘com.squareup.retrofit2:converter-gson:2.4.0’

// rx android

 implementation ‘io.reactivex.rxjava2:rxandroid:2.1.0’
 
 implementation ‘io.reactivex.rxjava2:rxjava:2.2.4’
 
 implementation ‘com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0’

if you have collection of Api like Bellow;
 
Ex. For get Color, API is “https://www.somesite.com/somepath/color";

 For get category, API is “https://www.somesite.com/somepath/category";
 
 For Post Email and Password, API is “https://www.somesite.com/somepath/login";

Now, Follow only 4 steps like below: 
 
1.First Create java Class named RetrofitClient:

import com.jakewharton.retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;

import okhttp3.OkHttpClient;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class RetrofitClient {

    private static Retrofit retrofit = null;
  
    public static Retrofit getClient(String baseUrl) {
        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                 .baseUrl(baseUrl)
                 .addConverterFactory(GsonConverterFactory.create())
          .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
          .build();
        }
        return retrofit;
    }
2.Create new java class named ApiUtils:

public class ApiUtils {

    private ApiUtils() {
    }

   public static final String BASE_URL = "https://www.somesite.com/somepath/";
//Base Url means portion of the full url which never change. only //end part will change
    public static APIService getAPIService() {
        return RetrofitClient
                .getClient(BASE_URL)
                .create(APIService.class);
    }

}


3. Now Create interface named APIService:


import io.reactivex.Observable;
import retrofit2.http.GET;


public interface APIService {

  @GET("category")
  Observable<CategoryResponseClass> getColors();

  @GET("color")
  Observable<ColorResponseClass> getCategory();

  @POST("login")
  @FormUrlEncoded
  Observable<LoginResponseClass>
  postLogin(@Field("email") String email,
  @Field("password") String password);
}
 
 
4.Last step, Calling Api to get response from yourActivity

package com.sesame.activities;

public class YourActivity extends AppCompatActivity {

    private APIService mAPIService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_your);


        //whenever you need colors from Api

        getAllColors();

        //whenever you need category from Api

        getAllCategory();

        // on Login Button Click

        loginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {

                processLogin()

            }
        });


    }

    private void getAllColors() {
        myprogressBar.show();
        mAPIService.getColors()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
new Observer<ColorResponseClass>() {

         @Override
         public void onSubscribe(Disposable d) {
                            }

         @Override
         public void onNext(ColorResponseClass colorResponseClass) {
//After getting Response Do your Stuff//

myprogressBar.dismiss();
                  processColor(colorResponseClass);
}

        @Override
        public void onError(Throwable e) {
        if (e != null) {
                                          Toast.makeText(YourActivity.this, e.getMessage(), Toast.LENGTH_SHORT).show();
          myprogressBar.dismiss();
                                }
           }

         @Override
         public void onComplete() {
              }
            }
         );

    }


    private void processSignIn() {
        String email = etEmail.getText().toString().trim();
        String password = etPassword.getText().toString().trim();
        if (email.equals("") || password.equals("")) {
            Toast.makeText(getApplicationContext(), "Email or Password should not blank", Toast.LENGTH_SHORT).show();
        } else {
            myprogressBar.show();
            mAPIService.postLogin(email, password)
                    .subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(
                            new Observer<LoginResponseClass>() {

           @Override
           public void onSubscribe(Disposable d) {
                                }

            @Override
           public void onNext(LoginResponseClass LoginResponseClass)        {
              //After getting Response Do your Stuff//
              myprogressBar.dismiss();
         if (LoginResponseClass.getStatus.equals("1"){
                                    
             //Login Successfull go to Next Activity//
}

             }

          @Override
          public void onError(Throwable e) {
          if (e != null) {
                                        Toast.makeText(YourActivity.this, e.getMessage(), Toast.LENGTH_SHORT).show();
myprogressBar.dismiss();
          }
       }

     @Override
      public void onComplete() {
                       }
                   }
              );
        }
    }

}
