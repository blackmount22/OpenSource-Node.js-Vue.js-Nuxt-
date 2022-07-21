### 1. bcrypt 패스워드 암호화 처리
---
```bash
npm i bcrypt //bcrypt 설치
```

회원 가입 함수 호출 시 패스워드 암호화 처리

```jsx
const bcrypt = require(’bcrypt’)

router.post('/', async(req, res, next) => {
	try{
		const hash = await bcrypt.hash(req.body.password, 10);
		// 이미 회원가입여부 체크
		const isUser = await db.User.findOne({
			where:{
				email:req.body.email
			},
			
		if(isUser){
				// 중략
		}

		const newUser = await db.User.create({
			email: req.body.email,
			password: hash, // HASH 암호화 된 패스워드 저장
			nickname: req.body.nickname
		});
	} catch (err) {
		console.log(error);
	}
});
```
<br>

### 2. 파일 업로드 (multer 사용)

```javascript
<input ref="imageInput" type="file" multiple hidden @change="onChangeImages">
<v-btn @click="onClickImageUpload" type="button">이미지 업로드</v-btn>

onClickImageUpload(){
    this.$refs.imageInput.click();
},
onChangeImages(e){
    console.log(e.target.files);    // 사진 고르면 e.target.files에 포함
    // 이미지는 FormData 형식
    const imageFormData = new FormData();
    [].forEach.call(e.target.files, (f) => {
        imageFormData.append('image', f);      // { image: [file1, file2] }
    });

    this.$store.dispatch('board/uploadImages', imageFormData);
}
```

이미지 업로드 시 FormData 형식으로 변경되어 업로드 된다. 하지만, 크롬 개발자 도구 Network > images > view Source 시 FomData의 형식을 보면 아래와 같이 **WebKitFormBoundary + 문자열** 형식이다. 

이처럼, 해석하기 어려운 FormData 형식을 위해 multer 패키지를 사용한다.

![image](https://user-images.githubusercontent.com/57103993/180215594-2ce403ad-06a3-4cc0-9ab9-50f8a4330cec.png)

```bash
npm -i multer  //multer 설치
```

---

- Back-end > multer 활용
    
    ```node.js
    const multer = require('multer');
    const path = require('path');
    
    const upload = multer({
    	storage : multer.diskStorrage{
    		destination(req, file, done) {
    			// 어디다 저장할지 정의
    			done(null, 'upload')
    			// 완료 시 (에러, 저장 위치) > 즉, uploads 폴더에 저장
    		},
    		filename(req, file, done) {
    			// 파일 명칭 정의
    			const ext = path.extname(file.originalname);
          const basename = path.basename(file.originalname, ext); // test.png, basename :test, ext = png
          done(null, basename + Date.now() + ext);    // 이름 + 시간(중복 방지를 위한) + 확장자
    		}
    	}, // 이미지를 어떻게 저장할 건지 명시
    	limit : {fileSize: 20 * 1024 * 1024}, //20MB 로 제한
    })
    ```
    

- 파일 실제 업로드 처리
    
    ```node.js
    router.post('/images', upload.array('image'), (req, res) => {
    	//req.files = []  업로드한 파일 정보가 배열로 저장
    	res.json(req.files.map(v => v.filename));
    })
    ```
<br>
