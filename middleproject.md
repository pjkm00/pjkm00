# 대학교 종합 정보 시스템
## 개요
- java를 이용한 대학교 학사 종합 정보 시스템

#### 만든 목적
- 학생이 수강 신청, 성적 확인, 과제 제출을 한 곳에서 가능
- 교수는 강의와 과제를 등록하고 학생 관리 가능

#### 일정
- 2023.05.25 ~ 2023.06.09
- 인원 : 3명

## 사용 기술 및 개발 환경
- Server : Apache Tomcat 8.5
- DB : Oracle11g
- Framework/Flatform : iBatis, Bootstrap, jQuery
- Language : JAVA, Javascript, HTML5, CSS3
- Tool : Eclipse, GitHub


## 내용
### 역할
PL, DB설계 및 구현, 로그인, 강의, 과제, 학사일정 관리

과제

1. 학생이 수강 신청한 강의의 과제목록이 보이고 과제 제출 여부 확인
<br>
StuAssignListAction.java 일부

~~~java
HttpSession session = req.getSession();
String stu_id = (String) session.getAttribute("stu_id");

List<AssignVO> assignList = null;
IAssignService service = AssignServiceImpl.getInstance();


assignList = service.getStuAssignList(stu_id);

for(AssignVO vo : assignList) {
  AssignVO assignVo = new AssignVO();
  assignVo.setAssign_no(vo.getAssign_no());
  assignVo.setStu_id(stu_id);
  int exist = 0;
  exist = service.existAssignFile(assignVo);
  if(exist != 0) {
    assignVo = service.getStuAssignFile(assignVo);
    vo.setAssign_path(assignVo.getAssign_path());
  }
}
~~~

assign.xml 일부
~~~xml
<select id="getStuAssignList" parameterClass="String" resultClass="assignVO">
		SELECT max(a.lec_code) lec_code
			 , max(a.assign_no)	assign_no
			 , max(assign_name)	assign_name
			 , max(assign_content)	assign_content
			 , max(assign_start)	assign_start
			 , max(assign_end)	assign_end
			 , max(lec_name)	lec_name
		FROM all_assignment a, all_lecturelist al, lecture l
		WHERE a.lec_code = l.lec_code
		AND a.lec_code = al.lec_code
		AND l.stu_id = #stu_id#
		GROUP BY a.assign_no
</select>
~~~

assignList.jsp 일부
~~~js
<%
if(request.getAttribute("assignList") == null){
%>
<tr class="odd" type="var" name="" style="text-align: center; height: 30px;">
  <td colspan="5">과제가 없습니다.</td>
</tr>	
<%
}else{
  String subState = "제출전";
  List<AssignVO> assignList = (List)request.getAttribute("assignList");
  Date todayd = new Date();
  
  for(AssignVO vo : assignList){
    int today = Integer.parseInt(fomatter2.format(todayd));
    int start = Integer.parseInt(fomatter2.format(vo.getAssign_start()));
    int end = Integer.parseInt(fomatter2.format(vo.getAssign_end()));
    
  
    if(vo.getAssign_path() != null && !"".equals(vo.getAssign_path())){
      subState = "제출완료";
    }else if(today <= end){
      subState = "제출전";
    }else{
      subState = "제출기간 초과";
    }
%>

~~~

<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/3d8b97fc-522c-433a-ac8e-3340e4f5d14a">

<br><br>
2. 교수가 자신이 등록한 과제를 제출한 학생 목록 조회하고 제출물 다운
<br>
SubAssignListAction.java 일부

~~~java
int assign_no = Integer.parseInt(req.getParameter("assign_no"));
String lec_code = req.getParameter("lec_code");

List<AssignVO> stuList = null;
List<AssignVO> assignList = null;

assignList = service.getLectureAssignList(lec_code);
stuList = service.getSubStudentList(assign_no);

req.setAttribute("lec_code", lec_code);
System.out.println(lec_code);
if(assignList != null) {
  
  req.setAttribute("assignList", assignList);
}
req.setAttribute("stuAssignList", stuList);
req.setAttribute("assign_no", assign_no);
~~~

submitAssignList.jsp 일부

~~~js
<%
String stu_grade = "";
for(AssignVO stu : stuList){
  
%>
<tbody>
<tr class="odd" type="var" id="<%=stu.getStu_id()%>" style="text-align: center; height: 30px;">
  <td><%=stu.getStu_id() %></td>
    <input type="hidden" name="stu_id" id="stu_id" value="<%=stu.getStu_id()%>">
  <td><%=stu.getStu_name() %></td>
  <td><%=stu.getDept_name() %></td>
  <td><%=fomatter.format(stu.getAssign_subdate()) %></td>
  <td><input type="button" value="다운로드" id="subAssignDownBtn" onclick="location.href='<%=request.getContextPath()%>/assign/subAssignDown.do?assign_path=<%=stu.getAssign_path()%>'"></td>
</tr>
<%
}
%>
</tbody>

~~~

SubAssignDownloadAction.java 일부

~~~java
String assign_path = req.getParameter("assign_path");
String assignName = assign_path.split("/")[1].trim();

String realPath = "C:/upload/assignFile/" + assign_path;

File file = new File(realPath);

if(file.exists()) {
  FileInputStream fin = null;
  OutputStream out = null;
  try {
    
    //ContentType설정
    res.setContentType("aplication/octet-stream; charset=utf-8");
    
    String encodedFileName = getFileNameEncoding(assignName, getBrowser(req));
    
    //Response의 Header content-disposition 속성 성정
    res.setHeader("Content-Disposition", "attachment; fileName=\"" + encodedFileName + "\"");
    
    //출력용 스트림 객체 생성 ==> Response 객체 이용
    out = res.getOutputStream();
    
    //파일입력용 스트림 객체
    fin = new FileInputStream(file);
    
    byte[] buffer = new byte[8192];
    int bytesRead = -1;
    
    //byte 배열을 이용해서 파일 내용을 읽어와 출력용 스트림으로 출력한다.
    while((bytesRead = fin.read(buffer)) != -1) {
      out.write(buffer, 0, bytesRead);
    }
    
    out.flush();
  }catch(IOException e){
    System.out.println("입출력 오류 : " + e.getMessage());
  } catch (Exception e) {
    
    e.printStackTrace();
  }finally {
    if(fin != null) fin.close();
    if(out != null) out.close();
  }
}else {	//파일이 없을경우의 처리
  res.setCharacterEncoding("UTF-8");
  res.setContentType("text/html; charset=utf-8");
  res.getWriter().println("<h3>파일이 존재하지 않습니다.</h3>");
}

~~~
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/914fce14-f19f-40aa-ab3f-64c3277f0220">

#### 산출물
<br>
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/37621377-b0a5-4986-8dfe-f3c65ef35fa6">
<br><br>
메뉴구조도
<br>
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/7f99f0a1-1cf7-408e-9307-abc59fe40052">
<br><br>
erd 
<br>
<img width="982" src="https://github.com/pjkm00/mink/assets/126858707/2b7f4894-ff25-4ff1-a833-0b3b69725713">
