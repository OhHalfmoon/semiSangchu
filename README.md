# semiSangchu
세미프로잭트 : 위치기반 중고거래 플랫폼 

프로잭트 수행인원 : 5명
프로잭트 소개 : 
 - 회원가입시 입력받은 주소를 통해 좌표값을 반환하여 판매자와의 거리를 산출한다.
 - 산출된 거리를 통해 자신의 특정 거리 반경 이내의 판매글을 확인 및 거래가 이루어진다.

담당역할 : 
1) 상품 거래 신청 수락 및 거절
2) 좌표값을 통한 거리 산출
3) 상품 거래 후기작성
 
담당역할 기능설명 :
1) 거래
 다른 쇼핑몰 프로잭트와는 달리, 중고물품거래 플랫폼이라는 특성상 사용자간 거래가 이루어진다는 특성이 있습니다.
그로인해 결제방법을
1 - 구매자가 플랫폼에 결제하면 플렛폼측에서 입금된 금액을 판매자에게 송금한다
2 - 결제에 관해서 플랫폼 측에서는 현재 물품의 판매 상태에 대한 정보만 개입한다
위의 두가지 방법이 초기에 논의되었고, 2번의 방법으로 개발이 진행되었습니다.
판매중, 거래중, 판매완료 3가지의 상태로 상태를 구분하였습니다.
판매중은 판매자가 어떠한 거래 요청에도 응하지 않은 상태입니다. 거래요청을 보낼 수 있습니다.
거래중은 판매자가 특정 요청에 대해 요청수락을 한 상태이며, 다수의 요청에 수락을 할 수 있고, 거래중인 상태에도 거래요청을 보낼 수 있습니다.
판매완료 상태는 물품이 거래완료된 상태이며, 거래요청을 보낼 수 없습니다.

사용자가 물품의 구매요청 버튼을 누릅니다. -> 판매자의 마이페이지에서 받은 요청을 확인 할 수 있습니다. 승인과 거절 2가지 버튼중 선택할 수 있으며, 승인을 누를 경우
물건의 상태값이 판매중에서 거래중으로 변경이 됩니다.
거래중의 단계에서 요청자들과 메세지를 통해 가격흥정등의 연락이 오고갈것이며, 최종적으로 한명의 요청자를 선택해 거래를 진행 후 거래완료 상태로 상태를 변경할 수 있습니다.

2) 위치기반
 회원가입을 할때 주소를 입력받습니다. 이때 주소검색은 다음지도에서 제공하는 api를 사용하여 검색이 이루어집니다.
자신의 주소에 해당하는 주소를 선택하면 그 주소에 대한 json값이 받아집니다. json값 중 사용자의 정확한 주소명을 반환받을 수 있었고, 이 정확한 주소명은 다시 카카오맵API에 입력되어 해당 주소의 위도와 경도값을 반환받을 수 있었습니다.
두 api를 통해 json데이터에서 회원의 주소, 동이름, 주소의 위도, 경도 4가지 정보를 각각 input hidden으로 받아와 DB에 저장합니다.
 회원가입때 받은 주소가 대표주소로 설정이되며, 마이페이지에서 최대 3개의 주소를 저장 및 관리할 수 있습니다. 주소 각각은 수정이 가능하며, 대표주소설정 역시 수정이 가능합니다.

처음 계획당시 거리계산은 JAVA Service를 통해 처리하려 했습니다. 하지만 한 메서드에서 여러번의 DB호출이 이루어지는 단점이 발생하였고, 이를 해결하기위해 DB에서 함수를 설정하였습니다. 거리를 구하는 공식은 구면좌표계를 활용하였습니다.
db 함수는 아래와 같습니다.
CREATE DEFINER=`SAMPLE`@`%` FUNCTION `db`.`get_distance2`(p_no bigint, yourx double, youry double) RETURNS double
begin 
	declare myx double;
	declare myy double;
	declare theta double;
	declare distance double;

	select x, y into myx, myy
	from `member` m join address a using(memberNo) where memberNo = p_no and a.mainAddr = 1;

	set theta = myx - yourx;
	
	set	distance = 60 * 1.1515 * (180/pi()) * acos(
				sin(myy * (pi()/180)) * sin(youry * (pi()/180)) +
				cos(myy * (pi()/180)) * cos(youry * (pi()/180)) * cos(theta*(pi()/180)));
	set distance = distance * 1.609344;
	return distance;
end
나의 좌표와 상품을 등록할때 설정한 주소의 좌표를 비교하여 거리를 산출합니다.
여러 값이 반환되기 때문에 List로 처리하며, 이후 화면에서 사용자는 원하는 거리반경을 정하고 (1,3,5km) 그 내부의 값에 해당하는 물품만 화면에 보이도록 하였습니다.

3) 후기
   후기는 물건의 상태값이 판매완료여야 하며, 그 판매된 물건의 거래내역에서 구매자에 해당하는 회원만 작성 및 수정을 할 수 있습니다.
후기는 사용자가 삭제하는것은 부당하다 생각하여(특히 판매자가 원치 않는 내용의 후기가 달렸다고 직접삭제하는것을 막고자 하였습니다) 삭제기능을 추가하지 않았습니다.
후기는 평점과 텍스트 후기 두가지를 남길 수 있습니다. 평점은 별 이모티콘을 클릭하여 1점부터 3점까지 1점단위로 줄 수 있습니다.
클릭된 별의 수를 parseInt를 통해 점수로 치환하여 화면에 같이 보이도록 하였습니다.
 구매자가 아닌 다른 회원들은 후기 입력창 자체가 보이지 않으며, 구매자만 후기 등록칸이 화면에 보이도록 조건을 걸었습니다.
단 조건의 문제로 인해 구매자는 작성된 후기가 보이는것이 아닌 계속 수정화면이 나오도록 설정되어있으며, 후기를 수정할 경우, 수정한 후기와 후기 수정 입력칸 2개가 동시에 화면에 보여집니다. 새로고침을 하면 다시 1개만 보이지만, 새로고침을 하지 않고 2개가 보이는 상태에서 후기를 다시 수정하여 저장할 경우 후기가 2개로 저장되는 버그가 있습니다.
버그는 아직 수정하지 못하였으며 후기 작성 및 수정을 해당화면에서 하는것이 아닌 모달창에서 입력받고, 물품 화면에서는 작성된 후기만 보이도록 하는것이 타당할것 같다는 생각이 들었습니다.

