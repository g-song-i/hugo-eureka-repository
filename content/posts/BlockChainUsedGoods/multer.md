---
title: Node.js - Multer를 이용해 안드로이드에서 이미지 업/다운로드
description: Node.js - Multer를 이용해 안드로이드에서 이미지 업/다운로드하기
toc: true
authors:
  - songi
tags:
  - Blockchain
  - Ethereum
  - Truffle
  - Smart contract
categories:
series: BlockChain Used Goods Trade Platform
date: '2020-05-20'
lastmod: '2020-05-20'
draft: false
---
</br>

## Content

이 부분은 이 중고거래 플랫폼을 만들면서 이더리움으로 고생했던 것만큼 오래 걸렸던 부분이에요. 파트너에게 부탁했는데 도저히 안되어서 넘겨받았는데 저도 힘들더라고요. 누군가 저에게 Multer 라이브러리가 아주 사용하기 쉽게 되어있다고 해서 금방 할 거라고 생각했는데 완전 오판이었습니다.
</br>
</br>
이 부분을 구현하는데 엄청난 도움을 준 블로그를 제일 아래에 첨부하도록 하겠습니다. 우선 통신에는 레트로핏 라이브러리를 사용합니다. 그래들 파일에 레트로핏을 implementation 해주세요.
</br>

```java
implementation 'com.squareup.retrofit2:retrofit:2.4.0'
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
implementation 'com.squareup.retrofit2:converter-scalars:2.4.0'
``` 

</br>
저는 이렇게 세개가 implementation 되어 있습니다. 다음으로 안드로이드에서 retrofit2.http.Multipart를 import 하는 ApiService 인터페이스를 구현해야 합니다. 아래는 ApiService.java입니다.
</br>
</br>

```java
package com.example.myapplication;

import okhttp3.MultipartBody;
import okhttp3.RequestBody;
import okhttp3.ResponseBody;
import retrofit2.Call;
import retrofit2.http.Multipart;
import retrofit2.http.POST;
import retrofit2.http.Part;

interface ApiService {
    @Multipart
    @POST("/upload")
    Call<ResponseBody> postImage(@Part MultipartBody.Part image, @Part("upload") RequestBody name);

}
```

</br>
이제 실제 업로드하는 java 코드를 볼 건데요. 주석과 토스트 메시지, 로그가 좀 많아요. 저희가 이 코드를 테스트로 구현하고, 실제 코드와 합쳤는데 실제 코드에서 변경해야 할 부분을 체크해 놓은 것이에요. 만약 이 코드를 적용하려면 저희가 체크해 놓은 부분을 다들 변경하실 것 같아 대부분의 주석은 일단 두겠습니다. 아래는 TestUploadImage.java입니다.
</br>
</br>

```java
package com.example.myapplication;

import android.annotation.TargetApi;
import android.app.Activity;
import android.app.AlertDialog;
import android.content.ClipData;
import android.content.ComponentName;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.content.pm.ResolveInfo;
import android.database.Cursor;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Color;
import android.net.Uri;
import android.nfc.Tag;
import android.os.Build;
import android.os.Bundle;
import android.os.Parcelable;
import android.os.PersistableBundle;
import android.provider.MediaStore;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.bumptech.glide.Glide;
import com.bumptech.glide.load.resource.bitmap.BitmapTransitionOptions;
import com.esafirm.imagepicker.features.ImagePicker;
import com.esafirm.imagepicker.features.ImagePickerComponentHolder;
import com.esafirm.imagepicker.features.ReturnMode;
import com.esafirm.imagepicker.features.imageloader.ImageLoader;
import com.esafirm.imagepicker.features.imageloader.ImageType;
import com.esafirm.imagepicker.model.Image;
import com.google.android.gms.common.util.ArrayUtils;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.yongbeam.y_photopicker.util.photopicker.PhotoPagerActivity;
import com.yongbeam.y_photopicker.util.photopicker.PhotoPickerActivity;
import com.yongbeam.y_photopicker.util.photopicker.utils.YPhotoPickerIntent;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.List;

//import io.reactivex.functions.Action;
import okhttp3.MediaType;
import okhttp3.MultipartBody;
import okhttp3.OkHttpClient;
import okhttp3.RequestBody;
import okhttp3.ResponseBody;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import retrofit2.Retrofit;

import static android.Manifest.permission.READ_EXTERNAL_STORAGE;
import static android.Manifest.permission.WRITE_EXTERNAL_STORAGE;
import static com.example.myapplication.URLS.URL_UPLOAD;

public class TestUploadImage extends AppCompatActivity {

    //이미지 뷰 5장
    ImageView image1, image2, image3, image4, image5;

    //이미지 업로드를 위해 상속한 인터페이스
    ApiService apiService;
    //유저 아이디 가져오기 위함
    User user;
    String user_id;
    private ArrayList<String> permissionsToRequest;
    private ArrayList<String> permissionsRejected = new ArrayList<>();
    private ArrayList<String> permissions = new ArrayList<>();
    private final static int ALL_PERMISSIONS_RESULT = 107;
    private final static int IMAGE_RESULT = 200;
    public final static int REQUEST_CODE = 1;

    //FloatingActionButton fabUpload;
    //Bitmap[] mBitmap = new Bitmap[5];
    ArrayList<Bitmap> mBitmap = new ArrayList<>();
    TextView textView;

    Button fabUpload, fab;

    Uri picUri;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_upload_image);

        askPermission();
        initRetrofitClient();

        image1 = (ImageView)findViewById(R.id.imageView1);
        image2 = (ImageView)findViewById(R.id.imageView2);
        image3 = (ImageView)findViewById(R.id.imageView3);
        image4 = (ImageView)findViewById(R.id.imageView4);
        image5 = (ImageView)findViewById(R.id.imageView5);

        fabUpload = findViewById(R.id.fabUpload);
        textView = findViewById(R.id.textView);
        fab = findViewById(R.id.fab);

        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //startActivityForResult(getPickImageChooserIntent(), IMAGE_RESULT);
                getPickImageChooserIntent();
            }
        });

        fabUpload.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                if (mBitmap != null) {

                    for (int i = 0; i < mBitmap.size(); i++)
                        multipartImageUpload(i);

                } else {
                    Toast.makeText(getApplicationContext(), "Bitmap is null. Try again", Toast.LENGTH_SHORT).show();
                }
            }
        });

        final PrefManager prefManager = PrefManager.getInstance(TestUploadImage.this);
        user = prefManager.getUser();

        if (prefManager.isLoggedIn()) {
            user_id = String.valueOf(user.getUser_id());
        }

        //initRetrofitClient();
    }

    private void askPermission() {

        permissions.add(WRITE_EXTERNAL_STORAGE);
        permissions.add(READ_EXTERNAL_STORAGE);
        permissionsToRequest = findUnAskedPermissions(permissions);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {

            if (permissionsToRequest.size() > 0)
                requestPermissions(permissionsToRequest.toArray(new String[permissionsToRequest.size()]), ALL_PERMISSIONS_RESULT);

        }

    }
   
    private void initRetrofitClient() {

        OkHttpClient client = new OkHttpClient.Builder().build();

        apiService = new Retrofit.Builder().baseUrl(URL_UPLOAD).client(client).build().create(ApiService.class);
    }

    //이미지 가져오기 
    public void getPickImageChooserIntent() {


        List<Intent> allIntents = new ArrayList<>();
        PackageManager packageManager = getPackageManager();
        ArrayList<Image> images = new ArrayList<>();
        //Intent intent = new Intent(Intent.ACTION_PICK);
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("image/*");
        intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true);
        //intent.setType(MediaStore.Images.Media.CONTENT_TYPE);
        startActivityForResult(Intent.createChooser(intent, "이미지 다중 선택"), REQUEST_CODE);

    }

    //이미지 촬영 후 Uri 근데 안쓸거 같지만 일단 놔두자
    private Uri getCaptureImageOutputUri() {

        Uri outputFileUri = null;
        File getImage = getExternalFilesDir("");
        if (getImage != null) {
            outputFileUri = Uri.fromFile(new File(getImage.getPath(), "profile.png"));
        }
        return outputFileUri;
    }

    //getPickImageChooserIntent()에서 startActivitiForResult하고 난 결과를 받는 메소드
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {

        super.onActivityResult(requestCode, resultCode, data);

        String imagePath = null;
        ArrayList<String> imageListUri = new ArrayList<>();
        ArrayList<Uri> realUri = new ArrayList<>();

        if (requestCode == REQUEST_CODE) {

            Toast.makeText(this, "여기 드가욤", Toast.LENGTH_SHORT).show();

            // photos = data.getStringArrayListExtra(PhotoPickerActivity.KEY_SELECTED_PHOTOS);
            if (data.getClipData() == null) {

                Toast.makeText(this, "다중 선택이 불가능한 기기입니다", Toast.LENGTH_SHORT).show();

            } else { //다중 선택 했을 경우

                ClipData clipData = data.getClipData();
                //Log.i("clipdata1 : ", String.valueOf(clipData.getItemCount()));
                //Log.i("clipdata2 : ", String.valueOf(clipData.getItemCount()));
                //Log.i("clipdata3 : ", String.valueOf(clipData.getItemCount()));


                if (clipData.getItemCount() > 5) {
                    Toast.makeText(TestUploadImage.this, "사진은 5장까지 선택 가능합니다", Toast.LENGTH_SHORT).show();
                } else if (clipData.getItemCount() == 1) {

                    Uri tempUri;
                    tempUri = clipData.getItemAt(0).getUri();
                    imagePath = tempUri.toString();

                    mBitmap.add(BitmapFactory.decodeFile(imagePath));
                    image1.setImageBitmap(mBitmap.get(0));
                    //File file = new File(imagePath);
                    //imageView1.setImageURI(Uri.fromFile(file));
                } else if ((clipData.getItemCount() > 1) && (clipData.getItemCount() <= 5)) {
                    for (int i = 0; i < clipData.getItemCount(); i++) {
                        Uri tempUri;
                        tempUri = clipData.getItemAt(i).getUri();
                        Log.i("temp: ", i + " " + tempUri.toString());
                        imageListUri.add(tempUri.toString());
                        realUri.add(tempUri);
                        //mBitmap.add(i, BitmapFactory.decodeFile(imageListUri.get(i)));

                        try {

                            mBitmap.add(i,BitmapFactory.decodeStream(getContentResolver().openInputStream(realUri.get(i))));

                        } catch (FileNotFoundException e) {
                            e.printStackTrace();
                        }
                    }

                    if(clipData.getItemCount() == 2) {
                        image1.setImageBitmap(mBitmap.get(0));
                        image2.setImageBitmap(mBitmap.get(1));

                    } if (clipData.getItemCount() == 3) {
                        image1.setImageBitmap(mBitmap.get(0));
                        image2.setImageBitmap(mBitmap.get(1));
                        image3.setImageBitmap(mBitmap.get(2));

                    } if(clipData.getItemCount() == 4) {
                        image1.setImageBitmap(mBitmap.get(0));
                        image2.setImageBitmap(mBitmap.get(1));
                        image3.setImageBitmap(mBitmap.get(2));
                        image4.setImageBitmap(mBitmap.get(3));

                    } if(clipData.getItemCount() == 5) {
                        image1.setImageBitmap(mBitmap.get(0));
                        image2.setImageBitmap(mBitmap.get(1));
                        image3.setImageBitmap(mBitmap.get(2));
                        image4.setImageBitmap(mBitmap.get(3));
                        image5.setImageBitmap(mBitmap.get(4));
                    }
                }
            }
        }
    }

    private String getImageFromFilePath(Intent data) {
        return getPathFromUri(data.getData());
    }

    public String getImageFilePath(Intent data) {
        return getImageFromFilePath(data);
    }

    private String getPathFromUri(Uri contentUri) {
        String[] proj = {MediaStore.Audio.Media.DATA};
        Cursor cursor = getContentResolver().query(contentUri, proj, null, null, null);
        int column_index = cursor.getColumnIndexOrThrow(MediaStore.Audio.Media.DATA);
        cursor.moveToFirst();
        return cursor.getString(column_index);
    }

    @Override
    public void onSaveInstanceState(@NonNull Bundle outState, @NonNull PersistableBundle
            outPersistentState) {
        super.onSaveInstanceState(outState, outPersistentState);

        outState.putParcelable("pic_uri", picUri);
    }

    @Override
    protected void onRestoreInstanceState(@NonNull Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);

        picUri = savedInstanceState.getParcelable("pic_uri");
    }

    private ArrayList<String> findUnAskedPermissions(ArrayList<String> wanted) {

        ArrayList<String> result = new ArrayList<String>();

        for (String perm : wanted) {
            if (!hasPermissions(perm)) {
                result.add(perm);
            }
        }

        return result;
    }

    private boolean hasPermissions(String permissions) {

        if (canMakeSmores()) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                return (checkSelfPermission(permissions) == PackageManager.PERMISSION_GRANTED);
            }
        }
        return true;
    }

    private void showMessageOKCancel(String message, DialogInterface.OnClickListener okListener) {

        new AlertDialog.Builder(this)
                .setMessage(message)
                .setPositiveButton("OK", okListener)
                .setNegativeButton("Cancel", null)
                .create()
                .show();
    }

    private boolean canMakeSmores() {
        return (Build.VERSION.SDK_INT > Build.VERSION_CODES.LOLLIPOP_MR1);
    }

    @TargetApi(Build.VERSION_CODES.M)
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        switch (requestCode) {

            case ALL_PERMISSIONS_RESULT:

                for (String perms : permissionsToRequest) {

                    if (!hasPermissions(perms)) {
                        permissionsRejected.add(perms);
                    }
                }

                if (permissionsRejected.size() > 0) {

                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                        if (shouldShowRequestPermissionRationale(permissionsRejected.get(0))) {
                            showMessageOKCancel("These permissions are mandatory for the application. Please allow access",
                                    new DialogInterface.OnClickListener() {
                                        @Override
                                        public void onClick(DialogInterface dialog, int which) {
                                            requestPermissions(permissionsRejected.toArray(new String[permissionsRejected.size()]), ALL_PERMISSIONS_RESULT);
                                        }
                                    });
                            return;
                        }
                    }
                }

                break;
        }

    }

    //실제 이미지를 업로드 하는 파트
    //여기서 파일의 이름을 바꿔야함
    private void multipartImageUpload(int index) {

        try {

            File filesDir = getApplicationContext().getFilesDir();
            //여기서 png 앞에를 유저 id + 레지스터 넘버 이런식으로 바꿀 것
            File file = new File(filesDir, filesDir.getName() + ".png");

            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            mBitmap.get(index).compress(Bitmap.CompressFormat.PNG, 0, bos);
            byte[] bitmapdata = bos.toByteArray();

            FileOutputStream fos = new FileOutputStream(file);
            fos.write(bitmapdata);
            fos.flush();
            fos.close();


            RequestBody reqFile = RequestBody.create(MediaType.parse("image/*"), file);
            MultipartBody.Part body = MultipartBody.Part.createFormData("upload", file.getName(), reqFile);
            RequestBody name = RequestBody.create(MediaType.parse("text/plain"), "upload");

            Call<ResponseBody> req = apiService.postImage(body, name);

            req.enqueue(new Callback<ResponseBody>() {

                @Override
                public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {

                    if (response.code() == 200) {
                        textView.setText("uploaded success");
                        textView.setTextColor(Color.BLUE);
                    }

                    Toast.makeText(getApplicationContext(), response.code() + "", Toast.LENGTH_SHORT).show();
                }

                @Override
                public void onFailure(Call<ResponseBody> call, Throwable t) {

                    textView.setText("uploaded fail");
                    textView.setTextColor(Color.RED);
                    Toast.makeText(getApplicationContext(), "req fail", Toast.LENGTH_SHORT).show();
                    t.printStackTrace();
                }
            });
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

</br>
실제 구현하신 분의 코드에 조금의 변형과 주석을 추가한 코드입니다. 기본적인 코딩에 대한 지식이 있다고 가정했기 때문에 거의 이해하실 것이라 생각합니다.
</br>
</br>
여기서 getCaptureImageOutputUri는 사진 촬영 후 Uri를 가져오는 메서드인데, 저희의 앱에서는 사진 촬영 없이 기존 앨범 이미지를 가져오기만 합니다. 그래서 해당 부분은 사용하지 않습니다. 또한 실제 전송 전에는 askPermission()으로 퍼미션을 확인해야 하고, initRetrofitClient()를 이용해 레트로핏 클라이언트를 초기화해야 합니다. 
</br>
</br>
getPickImageChooserIntent()를 이용해서 이미지를 선택해오게 되는데요, 이때 EXTRA_ALLOW_MULTIPLE 속성을 이용하여 다중 선택을 할 수 있습니다. 해당 메서드 마지막에서 startActivityForResult를 호출하게 되면 해당 메서드를 onActicityResult에서 결과로 받습니다. onAcitivityResult에서 이 결과를 어떻게 처리할 건지 오버라이딩하면 됩니다. 제 코드에 있는 수많은 if문은 머쓱하니 지나가 주세요ㅠ. Result를 처리하는 함수에서 비트맵을 초기화합니다.
</br>
</br>
실제로 upload는 multipartImageUpload에서 진행합니다. 이미지를 가져와서 비트맵이 있는 만큼 업로드를 하게 됩니다. 그러면 이제 Node.js 서버로 요청하게 되는데, 서버에서 이를 처리하는 코드를 보여드리겠습니다. 아래는 server.js입니다. 
</br>
</br>

```java
var _storage = multer.diskStorage({

	/*
	destination: function (req, file, callback) {

		//case: file type is image
		if(file.mimetype == "image/jpg" || file.mimetype == "image/jpeg" || file.mimetype == "image/png") {
			console.log("image");
			callback(null, "uploads/");
		} else {
			console.log("not image");
		}
	},
	filename: function (req, file, callback) {
		callback(null, register_number + "_" + file.originalname);
	}
	*/

	destination: 'uploads/',
	filename: function(req, file, cb) {
		return crypto.pseudoRandomBytes(16, function(err, raw) {
			if(err) {
				return cb(err);
			}
			//return cb(null, ""+(raw.toString('hex')) + (path.extname(file.originalname)));
			return cb(null, file.originalname);
		});
	}
});

//업로드
app.post('/upload', 
	multer({
		storage: _storage
	}).single('upload'), function (req, res) {

	try {

		let file = req.file;
		//const files = req.files;
		let originalName = '';
		let fileName = '';
		let mimeType = '';
		let size = 0;

		if(file) {
			originalName = file.originalname;
			filename = file.fileName;//file.fileName
			mimeType = file.mimetype;
			size = file.size;
			console.log("execute"+fileName);
		} else{ 
			console.log("request is null");
		}

	} catch (err) {

		console.dir(err.stack);
	}

	console.log(req.file);
	console.log(req.body);
	res.redirect("/uploads/" + req.file.originalname);//fileName

	return res.status(200).end();

});

app.get('/uploads/:upload', function(req, res) {


	var file = req.params.upload;
	console.log(file);
	var img = fs.readFileSync(__dirname + "/uploads/" + file);

	res.writeHead(200, {'Content-Type': 'image/png'});
	res.end(img, 'binary');
});

 /* 다운로드 요청 처리 */
var register_number_for_img;
var img_cnt;

app.post("/getimgmain", function(req, res){

	console.log("이미지요청");
	console.log(req.body);
	//var register_number = req.body.register_number;
	var register_number = req.body.register_number;
	var i=req.body.img_cnt;
		var filename = register_number+"_"+i+".png";
		var filePath = __dirname + "/uploads/" + filename;

		fs.readFile(filePath,
			function (err, data)
            {	
				if(err){
					console.log(err);
					filename = "defaltImg.png";
					filePath = __dirname + "/uploads/" + filename;
					fs.readFile(filePath,
						function (err, data)
						{
							console.log(filePath);
							console.log(data);
							res.end(data);
						});

				}else{
				console.log(filePath);
				console.log(data);
				res.end(data);
				}				

            }
		);
	
		
  });

  app.post("/getimg", function(req, res){

	console.log("이미지요청");
	console.log(req.body);
	var register_number = req.body.register_number;
	var i=req.body.img_cnt;
		var filename = register_number+"_"+i+".png";
		var filePath = __dirname + "/uploads/" + filename;

		fs.readFile(filePath,
			function (err, data)
            {	
				if(err){
					console.log(err);
					res.end(null);

				}else{
				console.log(filePath);
				console.log(data);
				res.end(data);
				}				

            }
		);
	
		
  });
```

</br>
먼저 사용할 storage를 만들어주어야 합니다. storage의 주석은, 제가 코딩한 걸 파트너가 원래 코드에 합치는 작업을 하면서 주석 처리했습니다. 검색해보면 저렇게 사용하시는 분이 많아서 참고하시라고 일단 놔둡니다. 기본적인 자바 스크립트를 아신다는 가정 하에 아마 거의 다 이해하실 거라 생각합니다. 이해 안 되는 부분 댓글 남겨주세요!
</br>
</br>
이미지 요청을 처리하는 부분이 두 군데인 이유는, 저희 앱에서 사진을 여러 장 등록할 수 있지만 메인에서 보이는 이미지는 제일 처음 이미지, 혹은 디폴트 이미지입니다. 그래서 메인에서 이미지를 보여주는 걸 처리하는 function 하나, 내부에서 이미지를 모두 보여주는 걸 처리하는 function 하나 총 두 개입니다.
</br>

## REFERENCE
https://www.journaldev.com/23709/android-image-uploading-with-retrofit-and-node-js

## Recent Comment
약간... 주석으로 수치플 당하는 느낌인데... 제가 다 읽기 힘들어서 일단 놔두는걸로 하겠습니다.