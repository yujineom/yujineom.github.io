---
layout: post
title:  "간단한 File Upload 구현하기 - 초보주의 (Spring Boot, Javascript)"
date:   2019-01-30
author: yujin eom
categories: spring javascript
---

메일 서비스를 개발하면서, 개발 1주차에 메일 쓰기를 담당하게 되었다.  
메일의 기본적인 기능 중, 파일을 첨부할 수 있어야 하니 파일 업로드 기능도 함께 구현하게 되었다.  
처음엔 파일업로드 관련 라이브러리를 사용하려고 했는데, 하다보니 '스스로 해보자' 라는 알 수 없는 욕심에 이끌려 직접 구현하게 되었다.

<br><br>
__그렇다면 너무나도 간단하고 심플한 파일 업로드를 구현해보자!__
<br><br>
먼저 파일을 첨부할 수 있도록 input 태그를 추가한다.

```html
<input type="file" name="file" id="file-element" multiple>
```

multiple 속성을 추가하면, 여러 파일을 한 번에 선택할 수 있다.
[참고문서](https://developer.mozilla.org/ko/docs/Web/HTML/Element/Input/file)

<br><br>
그리고 나는 첨부된 파일들의 리스트를 보여주고 싶었다. 첨부된 파일 중 일부를 삭제할 수 있도록 리스트마다 x 버튼 같은 것도 넣어주고~(생각보다 고려해야할 것이 많았다 ;-<)

그럼 위에서 선언한 input 태그에 change 이벤트를 걸어준다. 파일이 선택되었을 때 li 리스트 추가하는 작업, 파일의 사이즈들을 합하여 계산하는 작업 등등을 수행할 것이다.
<br><br>



```javascript
var storedFiles = [];

element.addEventListener('change', function () {        
  var fileList = this.files; 
  for (var i = 0; i < fileList.length; i++) {
    ... 생략
    storedFiles.push(fileList[i]);
    ul.appendChild(li);
  }
});
```

**this.files** 로 선택된 파일을 가져올 수 있다.

(name, lastModified, size 등등의 정보를 담고있다. 위의 참고문서에서 나보다 더 친절한 설명을 볼 수 있음)

storedFiles에는 첨부된 파일들을 쭉 담아놓는다.

처음에는 input 태그로 파일을 첨부하면 선택된 파일들이 계속 쌓이는 줄 알았다. input 태그로 또 다시 파일을 선택하면 이전의 file 데이터들은 다 날아간다... 그래서 storedFiles에 files로 받아온 것들을 쭉 담아준다!

li 에 파일명, 사이즈 속성을 이용하여 내용들을 만들어주고 ul에 추가! (이건 각자 파일리스트 ui에 맞게 만들어주면 될 것 같다.)

이렇게 하면 간단하게 파일들을 첨부하고, 첨부된 파일 리스트들을 볼 수 있다.
나는 첨부된 파일들을 삭제하기 위해 li 태그내에 삭제 버튼도 함께 넣어주었다.

<br><br><br>
그렇다면 파일리스트에서 특정 파일을 삭제해보자.
간단하게 li 태그내에 위치한 삭제 버튼에 대하여 click 이벤트를 달아준다.


```javascript
btnRemove.addEventListener('click', function () {
  var index	 = li.index();
  storedFiles.splice(index, 1);
  // 해당 li 태그 제거
  ... 생략
});
```

삭제 버튼이 속한 li 태그를 찾고, ul 내에서 해당 li의 index의 파일을 storedFiles에서 제거해준다. 



<br><br><br>
그럼 이제 첨부파일을 포함한 메일 내용들을 실제로 submit 해보자.

```javascript
var formData = new FormData();

for(var i=0;i<storedFiles.length;i++) {
  formData.append("files", storedFiles[i]);
}
```
[참고문서](https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData)  
전송 전, storedFiles에 담긴 내용들을 FormData에 추가해준다.  


```javscript
var formData = new FormData();

formData.append("files", storedFiles);
```
위와 같이 FormData에 array를 그대로 담은 경우, 뒤에 나올 Controller예제에서 parameter로 인식하지 못한다.  
이유는 아직까지 잘 모르겠지만, 이것때문에 많이 헤맸다.
<br><br>
__그럼, 이제 전송된 데이터들을 저장해볼까?__
<br>
* * *



```java
@PostMapping("/upload")
public ResponseEntity<?> uploadFile(@RequestParam("files") MultipartFile[] files) {
    logger.debug("Multiple file upload!");

    // Get file name
    String uploadedFileName = Arrays.stream(uploadfiles).map(x -> x.getOriginalFilename())
            .filter(x -> !StringUtils.isEmpty(x)).collect(Collectors.joining(" , "));

    if (StringUtils.isEmpty(uploadedFileName)) {
        return new ResponseEntity("please select a file!", HttpStatus.OK);
    }

    try {
        saveUploadedFiles(Arrays.asList(uploadfiles));
    } catch (IOException e) {
        return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }

    return new ResponseEntity("Successfully uploaded - "
            + uploadedFileName, HttpStatus.OK);

}

//save file
private void saveUploadedFiles(List<MultipartFile> files) throws IOException {

    for (MultipartFile file : files) {
        if (file.isEmpty()) {
            continue; //next pls
        }

        byte[] bytes = file.getBytes();
        Path path = Paths.get(UPLOADED_FOLDER + file.getOriginalFilename());
        Files.write(path, bytes);
    }
}
```
MultipartFile[]로 FormData에 담은 파일정보들을 담아온다.
그럼 이제 담아온 파일들은 특정 경로에 file을 생성한다.  
위의 소스는 [이 포스트](https://www.mkyong.com/spring-boot/spring-boot-file-upload-example-ajax-and-rest/)를 참고하였다.  
ajax로 Spring 파일 업로드 예제가 너무나도 잘 되어있다!

<br><br><br><br><br>
**느낀점**
* * *
이래저래 많이 생략하고 딱 핵심만 담은 부분들을 글로 작성하였다. 직접 구현하다보니 일일이 신경써야될 부분도 많고, 실제로 놓치고 지나간 부분도 많은 것 같다. 일단 1주차 개발에서 파일 업로드는 이 정도로 완성했지만 부족한 점이 많은 것 같다.

현재는 선택된 파일을 한 번에 전송하여 전송된 파일들을 일일이 저장하다보니 파일의 갯수와 용량이 조금 커지면 업로드까지의 시간이 조금 오래 걸린다 ;-<

비록 아직까진 비루하지만, 앞으로 남은 개발기간동안 어떻게 개선될지 기대된다!
