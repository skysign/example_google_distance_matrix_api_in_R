시작하기전 준비
===============

구글 Distan Matrix API는 다른 OpenAPI를 사용하는 방법과 비슷합니다.

-   R에서 국민연금 가입현황 OpenAPI 사용하기 참고
-   <a href="https://github.com/skysign/example_korea_NPS_openapi" class="uri">https://github.com/skysign/example_korea_NPS_openapi</a>

구글 클라우드 플랫폼 -&gt; ‘사용자 인증정보’ 에서 키를 만들어 주세요.

두점 그리고 시간
----------------

구글 Distan Matrix API는 출발점에서 부터 시작해서, 도착점에 다다를 때
까지 교통수단 별 걸리는 시간을 알려주는 API입니다. 따라서, 입려값으로,
출발점과, 도착점 2개의 점이 필요합니다.

그리고, 하나더!, 시간이 필요합니다. 교통 상황은 시간에 따라서 큰 영향을
받습니다. 따라서, 도착시간 또한 입력해 주어야 합니다.

함수로 만들기
=============

google\_distance\_matrix라는 이름으로 함수를 만들어 봤습니다. \*
src\_addr : 출발지, 한글 주소명 사용가능합니다 \* dst\_addr : 도착지,
한글 주소명 사용가능합니다 \* arrival\_time : 도착 시간, unix time으로
계산 해 줘야 합니다. \* key : API Key

    library(httr)
    library(XML)

    google_distance_matrix <- function(src_addr,
                                       dst_addr,
                                       arrival_time,
                                       key) {
      # myurl = 'https://maps.googleapis.com/maps/api/distancematrix/json'
      myurl = 'https://maps.googleapis.com/maps/api/distancematrix/xml'
      
      res <- httr::GET(
        url = myurl,
        query = list(
          origins = src_addr,
          destinations = dst_addr,
          language = 'ko',
          mode = 'transit',
          arrival_time = arrival_time,
          key = key
        )
      )

      res_xml = httr::content(res, as = 'text')
      return(res_xml)
    }

도착 시간
---------

getSecondsOfUTCUntilTodayHHMMKST() 함수는 앞에서 만든
google\_distance\_matrix함수의 도착시간을 계산 하기 위한 함수 입니다.

-   오늘 연/원일 에서 기준으로 HH 시간, MM 분을 입력 받아서
-   GMT+9 기준 datetime으로 만든 다음
-   GMT(UTC)로 변환 합니다.

<!-- -->

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    getSecondsOfUTCUntilTodayHHMMKST <- function(HH, MM) {
      now<-strptime(now(),"%Y-%m-%d %H:%M:%S")
      now
      # now$year
      # now$mon
      # now$mday
      
      year = now$year + 1900
      month = now$mon + 1
      day = now$mday
      
      HHMMSS = sprintf(' %d:%d:%d', HH, MM, 00)
      
      str_date = paste(year, month, day, HHMMSS)
      #str_date = paste(year, month, day, ' 10:30:00')
      # str_date
      
      # a = strptime(str_date,"%Y %m %d %H:%M:%S", tz = 'Etc/GMT-9')
      # today_HHMM_1930_KST = as.POSIXct(a, tzone = 'Etc/GMT-9')
      
      a = strptime(str_date,"%Y %m %d %H:%M:%S", tz = 'Asia/Seoul')
      today_HHMM_KST = as.POSIXct(a, tzone = 'Asia/Seoul')
      # today_HHMM_KST

      HHMM_UTC = as.POSIXct(as.integer(today_HHMM_KST), tz = 'GMT', origin = '1970-01-01')
      # HHMM_UTC
      
      return(as.integer(HHMM_UTC))
    }

    getSecondsOfUTCUntilTodayHHMMKST(19, 30)

    ## [1] 1568197800

google\_distance\_matrix 함수 실행해 보기
-----------------------------------------

    key = readLines('google_distance_matrix_key.private', n =1)
    key

    ## [1] "AIzaSyAp4QBHZ0FMwcSo7v5MIAea_3qdNu89wI0"

    src_addr = '서울 종로구 청운동'
    dst_addr = '서울특별시 서초구 서초중앙로 83'
    res = google_distance_matrix(src_addr,
                           dst_addr,
                           getSecondsOfUTCUntilTodayHHMMKST(19, 30),
                           key)
    res

    ## [1] "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<DistanceMatrixResponse>\n <status>OK</status>\n <origin_address>대한민국 서울특별시 종로구 청운동</origin_address>\n <destination_address>대한민국 서울특별시 서초구 서초3동 서초중앙로 83</destination_address>\n <row>\n  <element>\n   <status>OK</status>\n   <duration>\n    <value>2983</value>\n    <text>50분</text>\n   </duration>\n   <distance>\n    <value>15980</value>\n    <text>16.0 km</text>\n   </distance>\n  </element>\n </row>\n</DistanceMatrixResponse>\n"

결과 XML로 파싱해보기기
-----------------------

    library(XML)

    xmlResult <- xmlParse(res)
    xmlRoot = xmlRoot(xmlResult)
    xmlRoot

    ## <DistanceMatrixResponse>
    ##   <status>OK</status>
    ##   <origin_address>대한민국 서울특별시 종로구 청운동</origin_address>
    ##   <destination_address>대한민국 서울특별시 서초구 서초3동 서초중앙로 83</destination_address>
    ##   <row>
    ##     <element>
    ##       <status>OK</status>
    ##       <duration>
    ##         <value>2983</value>
    ##         <text>50분</text>
    ##       </duration>
    ##       <distance>
    ##         <value>15980</value>
    ##         <text>16.0 km</text>
    ##       </distance>
    ##     </element>
    ##   </row>
    ## </DistanceMatrixResponse>
