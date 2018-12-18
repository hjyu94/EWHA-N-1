<h1>"EWHA-N-1"</h1>
<h2>N분의 1</h2>
혼밥러들이 배달 음식을 먹는 방법!
배달비와 최소주문 금액이 부담되는 혼밥러들이 모여 
음식을 같이 주문하고 나누어 먹을 수 있도록 매칭-채팅 서비스를 제공하는 앱입니다.

<h2>1. 설치 방법 및 사용법</h2>
디바이스를 연결하여 앱을 실행 후 구글 아이디를 이용하여 로그인 하세요.
지도에 표시된 채팅방에 들어가 해당 음식점에서 먹고 싶은 메뉴를 정하거나,
먹고 싶은 음식이 없을 시 새로 채팅방을 만들 수 있습니다.
<h2>2. 주요 기능 코드/API 설명</h2>
  <h3>2.1 가격을 1/n으로 나눠주는 기능 </h3>
    <h4>2.1.1 총 가격 입력받기</h4>
![img1](https://user-images.githubusercontent.com/43066601/50151347-fe6f4100-0303-11e9-9731-f22ffa730462.PNG)


     ```java
     @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.dialog_setting_cancel:
                dialogListener.onNegativeClicked();
                break;

            case R.id.dialog_setting_ok:
                SettingModel setting = new SettingModel(
                    menu.getText().toString().trim(),
                    Integer.parseInt(price.getText().toString().trim()),
                    place.getText().toString().trim(),                            
                    hour+"시 "+minute+"분"
                );
                dialogListener.onPositiveClicked(setting);
                break;
        }
       }
      ```
    
 MessageSettingDialog.java의 onClick 함수에서에서 구현합니다. 사용자가 채팅방 정보를 입력하고 설정 을 누르면 (dialog_setting_ok 인 경우), SettingModel의 객체를 생성합니다. SettingModel.java는 채팅방의 정보를 저장하는 클래스입니다. 
       SettingModel 의 생성자는 
       
     
       `public SettingModel(String menu, int price, String place, String time)`
     
       
 입니다. price.getText()으로 사용자가 입력한 가격을 받아오고, 생성자를 이용해서 이를 SettingModel.java의 price 필드에 저장합니다.
  
   
      
  <h4>2.1.2 채팅방 인원수 구하기</h4>
  ChatRoom.java은 현재 채팅방을 나타내는 클래스 입니다.
   
   
       `public Map<String, Boolean> users = new HashMap<>();`
       
       
  ChatRoom.java에서 현재 채팅방에 있는 사용자를 HashMap 형태로 나타냅니다.
  
       ```java
       ChatRoom current_chatroom;
       current_chatroom.users.size()    //채팅방 인원수
       ```
 
<h4>2.1.3 1인당 가격 계산하기</h4>
```java
DatabaseReference chatroomRef = FirebaseDatabase.getInstance().getReference().child("chatroom");
DatabaseReference userRef = FirebaseDatabase.getInstance().getReference().child("users");
DatabaseReference settingRef = FirebaseDatabase.getInstance().getReference().child("Setting");
```
 채팅방 정보를 저장할 때 Firebase 실시간 데이터베이스를 이용합니다. 데이터베이스에서 데이터를 읽고 쓰려면 DabaseReference의 인스턴스가 필요합니다
 여기서 settingRef가 Setting의 하위 노드를 가리키고 있습니다. Setting이 채팅방 정보입니다
 
 파이어베이스 이미지 삽입
 <Firebase 실시간 데이터베이스>
 
 채팅방 메뉴 이미지 삽입
 <채팅방 정보>
 
 
 
        ```java
    settingRef.orderByKey().equalTo(key).addValueEventListener(new ValueEventListener() {
    @Override
    public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
        for (DataSnapshot item : dataSnapshot.getChildren()) {
            SettingModel setting = item.getValue(SettingModel.class);
            info_menu.setText(setting.menu);
            info_place.setText(setting.place);
            setting.price = setting.price/current_chatroom.users.size();
            info_price.setText(setting.price+"");
            info_time.setText(setting.time);
            Toast.makeText(MessageActivity.this, "메뉴가 설정되었습니다. 채팅방 정보를 확인하세요.", Toast.LENGTH_SHORT).show();
        }
    }
            ```
 ValueEventListener는 데이터베이스에서 일어나는 모든 변화를 감지합니다.            
 onDataChange() 메소드를 사용하여 이벤트 발생 시점을 기준으로 지정된 경로에 있는 내용의 정적 스냅샷을 읽을 수 있습니다. 이 메소드는 리스너가 연결될 때 한 번 호출된 후 하위를 포함한 데이터가 변경될 때마다 다시 호출됩니다. 이 함수를 이용해서 데이터가 변경될 때, item.getValue(SettingModel.class)로 SettingModel의 데이터를 가져옵니다.
 (1인분 가격)=(총 가격)/(채팅방 인원수)으로 1인당 가격을 구하고 info_price에 저장합니다.
             
             ```java
             info_price.setText(setting.price); 
             ```
        
info_price를 채팅방 정보에서 TextView로 나타내주면 됩니다
        
최종 이미지
    
 <h3>2.2 지도에 채팅방을 나타내는 마커 표시하기    </h3> 
 <h4>2.2.1 daum 지도 API </h4>
 ```java
 mapView = new MapView(this);
        mapView.setDaumMapApiKey("7f79f8a60fed7aca35c2e2638c658363");
        ViewGroup mapViewContainer = (ViewGroup) findViewById(R.id.map_view);
        mapViewContainer.addView(mapView);
  ```
 MapView 클래스 는 Daum 지도 화면을 보여주는 view class 입니다.
위 코드를 통해서 mapview를 띄웁니다.

 
<h4>지도에 마커 표시 </h4>
 `private void setMarkerOnMap(ChatRoom chatroom) { }`
setMarkerOnMap 함수는 위도와 경도를 받아서 해당 위치에 마커를 표시하는 함수입니다.

          ```java
          double latitude = chatroom.latitude;       //위도 
          double longitude = chatroom.longitude;      //경도
          String title = chatroom.title;             //채팅방 제목
          ```
 `    MapPoint mapPoint = MapPoint.mapPointWithGeoCoord(latitude, longitude);` 로 마커를 추가합니다.
 
```java
    if (chatroom.status.equals("yet")) {
        marker.setMarkerType(MapPOIItem.MarkerType.BluePin);
        marker.setSelectedMarkerType(MapPOIItem.MarkerType.RedPin);
    } else if (chatroom.status.equals("full")) {
        marker.setMarkerType(MapPOIItem.MarkerType.YellowPin);
        marker.setSelectedMarkerType(MapPOIItem.MarkerType.YellowPin);
    }
 ```
 만약 chatRoom의 인원 수가 꽉 찬 경우라면("full"), 마커가 노란색으로 표시되고 들어갈 채팅방으로 들어갈 수 없습니다.
인원수가 차지 않은 경우("yet"), 선택하기 전에는 마커가 파란색으로 표시되고, 선택 후에는 빨간색으로 표시됩니다.

`markers.add(marker);` 마커들을 Makers 배열에 저장합니다.
`mapView.addPOIItem(marker);`  지도에 마커를 붙입니다.

<h2>개발자 정보</h2>
<ul>
<li>
유효정(hjyu94): 
  <ul><li>지도 관련 기능 구현</li>
    <li>메인 화면에 지도 띄우기</li>
    <li>지도 클릭해서 채팅방 만들기</li>
    <li>채팅방 만든 후 마커 표시하기</li>
    <li>채팅방의 인원수 정보에 따라 클릭 가능한 마커인지 구분하여 지도에 마커 찍기</li>
    <li>각 메뉴별로 마커 모아보기</li>
    <li>채팅방 설정 기능 - 설정 버튼 클릭 후 메뉴, 장소, 시간 정하기</li>
    <li>채팅방 나가기 기능</li>
    <li>GPS를 이용하여 현재 위치 표시하기</li>
    <li>로그아웃 기능</li></ul>
</li>
<li>
  윤하은(Water-Silver): <ul><li>파이어베이스 - 구글 로그인, 채팅방 데이터베이스 연동</li>
  <li>구글 아이디를 이용하여 사용자를 식별하는 기능</li>
  <li>지도의 위치 정보를 받아와 채팅방 생성시 채팅방 이름과 위치 등의 정보를 저장 및 불러오기</li></ul>
</li>
<li>
  서현주(seohsj): <ul><li>데이터베이스</li>
  <li>메뉴판 데이터베이스 만들기</li></ul>
</li>
<li>
  최윤영(ILoveSpongebob) : <ul><li>채팅방안의 데이터베이스 </li>
  <li>채팅방 들어가면 메뉴창에 음식점 버튼 구현</li></ul>
</li>
</ul>
<h2>라이센스 정보</h2>
LGPL
