# [영화 예매 사이트](ttps://github.com/jjgod66/dw-final-group2)
## 개요
- 전자정부프레임워크를 이용한 영화 예매 사이트
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/d055ec25-c505-49b7-b596-3a203b64e2ff">

#### 일정
- 2023.06.30 ~ 2023.08.15
- 인원 : 6명

## 사용 기술 및 개발 환경
- Server : Apache Tomcat 8.5
- DB : Oracle11g
- Framework/Flatform : Spring MVC, SpringSecurity, myBatis, Bootstrap, jQuery, eGovFrame
- Language : JAVA, Javascript, HTML5, CSS3
- Tool : Eclipse, GitHub


## 내용
### 역할
DB설계, 고객 페이지의 대부분(예매, 영화, 스토어, 고객센터, 극장)

1. 아임포트 API를 사용하여 결제와 환불기능

결제(이니시스, 카카오 간편 결제 가능)
~~~js
function requestPay() { 
	let totalPrice = $('.totalPrice').text();
	let method = $('input[name="payMethod"]:checked').prop('id');
	let movie_name = '${mapData.MOVIE_NAME}';
	let buyer_name = '<%=member.get("NAME")%>';
	let buyer_tel = '<%=member.get("PHONE")%>';
	let buyer_email = '<%=member.get("EMAIL")%>';
	let price = ($('#totalpp').text()).replace(',', '');
	let discount = $('#disprice').text().replace(',', '');
	$('input[name="discount"]').val(discount);
	let merchant_uid = new Date().getTime();
	if(price <= 0){
		$('#payForm').prop('action', '<%=request.getContextPath()%>/reservation/pay0ResultRedirect.do');
		$('#payForm').append('<input type="hidden" name="merchant_uid" value="' + merchant_uid + '">');
		$('#payForm').submit();
		return;
	}
	
    IMP.request_pay({
        pg: method,
        pay_method: 'card',
        merchant_uid: merchant_uid,
        name: movie_name,
        amount: price,
        buyer_email: buyer_email,
        buyer_name: buyer_name,
        buyer_tel: buyer_tel,
    }, function (rsp) { // callback
        if (rsp.success) {
            console.log(rsp);
			$('input[name="json"]').val(JSON.stringify(rsp));
			alert("결제완료");
			$('#payForm').submit();	
			
        } else {
            console.log(rsp);
            pay_info = rsp;
            alert('결제가 실패 되었습니다.');

        }
    });
}
~~~

<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/0184985b-799d-48ce-bced-9f8a0a4f49c9">
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/a1d4a9ff-710c-4ae9-8661-dbf9bd180adb">
<br><br>

환불 시 필요한 토큰 받아오는 메서드
<br><br>
- imp_key와 imp_secret을 JsonObject로 보내려니 계속 오류가 발생
- JsonObject가 아닌 Map에 넣고 Gson을 사용해 Json으로 바꿔 보내니 토큰 받아짐

~~~java
@Override
public String getToken() {
  
  //서버로 요청할 Header
  HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
  
    
    Map<String, Object> map = new HashMap<>();
    map.put("imp_key", "키");
    map.put("imp_secret", "시크릿키");
    
   
    Gson gson = new Gson();
    String json=gson.toJson(map);
  //서버로 요청할 Body
   
    HttpEntity<String> entity = new HttpEntity<>(json,headers);
  return restTemplate.postForObject("https://api.iamport.kr/users/getToken", entity, String.class);
}

~~~

imp_uid로 환불하는 메서드
~~~java
public String refundSer(String imp_uid) {
  String result = "";
  try {
    String token = this.getToken();
    if(token == null) {
      throw new Exception();
    }
    String[] tokenList = token.split("\"");
    token = tokenList[9];
    System.out.println(token);
    
    HttpHeaders headers = new HttpHeaders();
    headers.add("Authorization", token);
    
    Map<String, Object> map = new HashMap<>();
    map.put("imp_uid", imp_uid);
    
    Gson gson = new Gson();
      String json=gson.toJson(map);
       
    HttpEntity<Map<String, Object>> entity = new HttpEntity<>(map, headers);
    String refund = restTemplate.postForObject("http://api.iamport.kr/payments/cancel", entity, String.class);
    

    System.out.println(refund);
    result = refund;
    
    
    
  } catch (Exception e) {
    e.printStackTrace();
    System.out.println("환불에 실패했습니다.");
    result = "F";
  }
  
  return result;
  
}

~~~

<br><br>
2. cropper.js와 html5canvas 라이브러리를 사용한 포토티켓 기능

<br>
이미지 파일을 선택하면 이미지를 편집할 수 있는 모달이 뜸
<br>

~~~js
function setThumbnail(event){
	var reader = new FileReader();

    reader.onload = function(event) {
      var img = $('<img>');
      img.prop("src", event.target.result);
      img.css('object-fit', 'cover');
      img.css('margin', '0 auto');
      img.prop('id', 'image');
      $(".them_img").html(img);
    };

    reader.readAsDataURL(event.target.files[0]);
    $('#photo-modal').modal('show');
};

~~~
<br>
모달에서 자르기 버튼 클릭 시 cropper가 실행되어 지정해둔 포토티켓의 비율에 맞게 사진 편집 가능
<br>

~~~js
var cropper;
$('#crop').on('click', function(){
  $(this).css('display', 'none');
  $('#success').css('display', '');
  
  let image = $('#image');
  cropper = image.cropper( {
    dragMode: 'move',
    viewMode:1,
    aspectRatio: 0.647,
    minCropBoxWidth:200,
      });
  
  })
~~~

<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/5e4f5de6-0981-4040-b933-a1882d110e5f">
<br><br>
완료 버튼 클릭 시 모달이 꺼지고 자른 이미지 저장
<br>

~~~js
 $('#success').on('click', function(){
    var image = $('#image');
    var result = $('#result');
    var canvas;
    if($('input[type="file"]').val() != ""){
    canvas = image.cropper('getCroppedCanvas');
    result.css('background-image','url(' + canvas.toDataURL("image/jpg") + ')');

    canvas.toBlob(function (blob) {
      var formData = new FormData();
    
      formData.append('croppedImage', blob);
      $.ajax({
        url : '<%=request.getContextPath()%>/photoTicket/uploadImg.do',
            method: 'POST',
              data: formData,
              processData: false,
              contentType: false,
              success: function (res) {
                console.log(res);
                $('#photo-modal').modal('hide');
                $('.them_img').html('');
                $('#inputfile').val('');
                $('#imgupbtn').css('display', 'none');
                $('#crop').css('display', '');
              $('#success').css('display', 'none');
              $('input[name="front_path"]').val(res);
              },
              error: function (err) {
                alert(err.status);
              },
      });
    })
    }else{
        alert('사진을 업로드 해주세요');
        $('input[type="file"]').focus();
        return;
    }
 })
~~~

~~~java
/* 파일 유무 확인 */
if(!(multi == null || multi.isEmpty() || multi.getSize() > 1024 * 1024 * 1)) {
  /*파일 저장 폴더 설정	*/
  uploadPath = photoTicketUploadPath + File.separator + uploadPath;
  fileName = UUID.randomUUID().toString().replace("-", "") + ".jpg";
  File storeFile = new File(uploadPath, fileName);
  
  storeFile.mkdirs();
  
  //local HDD에 저장.
  multi.transferTo(storeFile);
}

~~~

<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/35ca1ffb-2d70-47a7-852c-ad79290db473">
<br><br>

3. 스프링 스케줄러 사용
<br><br>
- 스프링 스케줄러를 사용하여 예매한 영화의 상영 다음날 포인트가 적립
- 매일 자정에 생일인 회원에게 생일 쿠폰을 지급
- 매달 1일 전달에 예매 실적에 따라 등급을 변경하는 기능을 구현

~~~java
@Autowired
PointService pointService;

//상영 다음날 해당 영화 예매한 회원들한테 포인트 적립해주는 스케줄러
@Scheduled(cron = "0 0 0 ? * *")
public void insertMoviePoint() throws SQLException {
  pointService.insertMoviePoint();
}

@Autowired
private CouponService couponService;

//해당일에 생일인 회원에게 생일축하 쿠폰 넣어주는 스케줄러
@Scheduled(cron = "0 0 0 ? * *")
public void insertBirthCoupon() throws SQLException {
  couponService.insertBirthCoupon();
}

@Autowired
private MemberService memberService;

//매달 1일 전달에 예매를 10번이상 한 회원 VIP로 업그레이드 헤주는 스케줄러
@Scheduled(cron = "0 0 0 1 ? *")
public void updateMemgrade() throws SQLException {
  memberService.updateMemgrade();
}
~~~


## 산출물

#### erd 일부
영화관련
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/6ea04449-f7a8-481d-bdb1-06a844fc7433">
<br><br>
영화관관련
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/a4dee5ce-48b7-4756-978e-9382bd1ee42f">
<br><br>
예매관련
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/a6477608-71a9-460a-b496-bd0ff99b0844">
<br><br>
회원관련
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/ff91dc1a-3fee-46d1-b3aa-0c663a2b565d">
<br><br>
관리자관련
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/20f42f3e-36c6-4c90-afb2-fa3ed1e27d64">

