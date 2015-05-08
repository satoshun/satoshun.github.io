+++
date = "2015-02-22T00:00:00Z"
title = "様々な言語のフィボナッチ関数"
tags = ["android", "webapi"]
blogimport = true
type = "post"
draft = true
+++

## 目標

Android上で2点間の経路をディスプレイングする. 2点間の経路は徒歩(自転車)で行ける程度の距離を想定.


## 前提知識

Androidにnativeで2点間の経路をマッピングするメソッドはない. 独自実装 or サードパーティライブラリを使う必要がある.

Directions APIの基本的な使い方は, 2点のLat, Lngを指定して, http(s)でrequestを送る. 結果は XML or JSONで返される. 以下の形式のResponseが返される.

```
{
   "routes" : [
      {
         "bounds" : {
            "northeast" : {
               "lat" : -33.8660807,
               "lng" : 151.21814
            },
            "southwest" : {
               "lat" : -33.8849029,
               "lng" : 151.1950405
            }
         },
         "copyrights" : "Map data ©2015 Google",
         "legs" : [
            {
               "distance" : {
                  "text" : "3.9 km",
                  "value" : 3900
               },
               "duration" : {
                  "text" : "13 mins",
                  "value" : 781
               },
               "end_address" : "122 Flinders Street, Darlinghurst NSW 2010, Australia",
               "end_location" : {
                  "lat" : -33.8849029,
                  "lng" : 151.21814
               },
               "start_address" : "48 Pirrama Road, Pyrmont NSW 2009, Australia",
               "start_location" : {
                  "lat" : -33.8663021,
                  "lng" : 151.195935
               },
               "steps" : [
                  {
                     "distance" : {
                        "text" : "75 m",
                        "value" : 75
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 12
                     },
                     "end_location" : {
                        "lat" : -33.8660807,
                        "lng" : 151.195169
                     },
                     "html_instructions" : "Head \u003cb\u003ewest\u003c/b\u003e on \u003cb\u003eTrouton Pl\u003c/b\u003e toward \u003cb\u003eDarling Island Rd\u003c/b\u003e",
                     "polyline" : {
                        "points" : "joumEsmyy[ETCLG^CNKf@ENCL"
                     },
                     "start_location" : {
                        "lat" : -33.8663021,
                        "lng" : 151.195935
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "30 m",
                        "value" : 30
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 12
                     },
                     "end_location" : {
                        "lat" : -33.8663187,
                        "lng" : 151.1950338
                     },
                     "html_instructions" : "Turn \u003cb\u003eleft\u003c/b\u003e onto \u003cb\u003eDarling Island Rd\u003c/b\u003e",
                     "maneuver" : "turn-left",
                     "polyline" : {
                        "points" : "~mumEyhyy[HBF@F@HDBDFF"
                     },
                     "start_location" : {
                        "lat" : -33.8660807,
                        "lng" : 151.195169
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.4 km",
                        "value" : 399
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 74
                     },
                     "end_location" : {
                        "lat" : -33.8687358,
                        "lng" : 151.1978188
                     },
                     "html_instructions" : "At the roundabout, take the \u003cb\u003e1st\u003c/b\u003e exit onto \u003cb\u003ePirrama Rd\u003c/b\u003e",
                     "maneuver" : "roundabout-left",
                     "polyline" : {
                        "points" : "noumE}gyy[?A@A?A@A@A@??A@?@??A@?@?@?@?@?@??@@?@CBEDGBEDEROn@]dAe@HGXMFE~@e@HEFEPKXUNQNOBCJSFKLYN]H[DODO@K@K@G@I@I?I?e@?E?C@C?C@E@CBE"
                     },
                     "start_location" : {
                        "lat" : -33.8663187,
                        "lng" : 151.1950338
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.1 km",
                        "value" : 143
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 43
                     },
                     "end_location" : {
                        "lat" : -33.870002,
                        "lng" : 151.1978513
                     },
                     "html_instructions" : "\u003cb\u003ePirrama Rd\u003c/b\u003e turns slightly \u003cb\u003eright\u003c/b\u003e and becomes \u003cb\u003eMurray St\u003c/b\u003e",
                     "polyline" : {
                        "points" : "r~umEkyyy[?ABC@A@ABA@?B?DAPAR?zACT@F?J@F?J@H@XD"
                     },
                     "start_location" : {
                        "lat" : -33.8687358,
                        "lng" : 151.1978188
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.9 km",
                        "value" : 894
                     },
                     "duration" : {
                        "text" : "2 mins",
                        "value" : 120
                     },
                     "end_location" : {
                        "lat" : -33.8774074,
                        "lng" : 151.2002482
                     },
                     "html_instructions" : "Turn \u003cb\u003eleft\u003c/b\u003e onto \u003cb\u003eDarling Dr\u003c/b\u003e\u003cdiv style=\"font-size:0.9em\"\u003eGo through 1 roundabout\u003c/div\u003e",
                     "maneuver" : "turn-left",
                     "polyline" : {
                        "points" : "nfvmEqyyy[@CBEBEDENUFKLORQXS@?@ANEDCd@GTCT@bABr@D^@@?@?@?@@B?R@B?FBJ@NB?A@A@??A@?@A@?@?@?@??@@?@??@@??@@??@@@?@?@?@P@|@DH?N?R?B?HAD?P?T@T?^AVARCTAPE@?f@I@AB?\\GXG@?NEHA@AJC@?JEbA]TKNGbAe@xD{AvBeAJGFEFGBEZo@"
                     },
                     "start_location" : {
                        "lat" : -33.870002,
                        "lng" : 151.1978513
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.5 km",
                        "value" : 461
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 83
                     },
                     "end_location" : {
                        "lat" : -33.8811195,
                        "lng" : 151.2019689
                     },
                     "html_instructions" : "At the roundabout, take the \u003cb\u003e3rd\u003c/b\u003e exit and stay on \u003cb\u003eDarling Dr\u003c/b\u003e",
                     "maneuver" : "roundabout-left",
                     "polyline" : {
                        "points" : "xtwmEqhzy[?A?A?C?A?A?A?A@A?A?A@A?A@A?A@A?A@?@A?ABA?ABA@?@A@?@A@?@?B?@?@?@?@?B@@?@@@@B@?@@@@@?@@@?@@@?@NMVUXKZM~As@VINIB?@ABALE@AHCBAHENG`Cw@^I\\Ml@SHATClBo@"
                     },
                     "start_location" : {
                        "lat" : -33.8774074,
                        "lng" : 151.2002482
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.3 km",
                        "value" : 307
                     },
                     "duration" : {
                        "text" : "2 mins",
                        "value" : 106
                     },
                     "end_location" : {
                        "lat" : -33.8804197,
                        "lng" : 151.2050228
                     },
                     "html_instructions" : "Turn \u003cb\u003eleft\u003c/b\u003e onto \u003cb\u003eUltimo Rd\u003c/b\u003e",
                     "maneuver" : "turn-left",
                     "polyline" : {
                        "points" : "~kxmEiszy[q@{BQi@k@kBGSAGEKSq@ACSq@EQDUFm@TiB"
                     },
                     "start_location" : {
                        "lat" : -33.8811195,
                        "lng" : 151.2019689
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "81 m",
                        "value" : 81
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 25
                     },
                     "end_location" : {
                        "lat" : -33.8811193,
                        "lng" : 151.2047944
                     },
                     "html_instructions" : "Turn \u003cb\u003eright\u003c/b\u003e onto \u003cb\u003eGeorge St\u003c/b\u003e",
                     "maneuver" : "turn-right",
                     "polyline" : {
                        "points" : "rgxmEkf{y[pAXx@R"
                     },
                     "start_location" : {
                        "lat" : -33.8804197,
                        "lng" : 151.2050228
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.1 km",
                        "value" : 110
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 32
                     },
                     "end_location" : {
                        "lat" : -33.8815867,
                        "lng" : 151.2058334
                     },
                     "html_instructions" : "Turn \u003cb\u003eleft\u003c/b\u003e onto \u003cb\u003eRawson Pl\u003c/b\u003e",
                     "maneuver" : "turn-left",
                     "polyline" : {
                        "points" : "~kxmE}d{y[p@_B^_A@WHW"
                     },
                     "start_location" : {
                        "lat" : -33.8811193,
                        "lng" : 151.2047944
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.3 km",
                        "value" : 273
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 68
                     },
                     "end_location" : {
                        "lat" : -33.8828645,
                        "lng" : 151.208348
                     },
                     "html_instructions" : "Continue onto \u003cb\u003eEddy Ave\u003c/b\u003e",
                     "polyline" : {
                        "points" : "|nxmEmk{y[F[DUDMFQNe@Xq@`AuBTg@Xm@dAaC"
                     },
                     "start_location" : {
                        "lat" : -33.8815867,
                        "lng" : 151.2058334
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.1 km",
                        "value" : 128
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 24
                     },
                     "end_location" : {
                        "lat" : -33.8817274,
                        "lng" : 151.2085805
                     },
                     "html_instructions" : "Turn \u003cb\u003eleft\u003c/b\u003e onto \u003cb\u003eElizabeth St\u003c/b\u003e",
                     "maneuver" : "turn-left",
                     "polyline" : {
                        "points" : "zvxmEe{{y[c@GeBSwAQ"
                     },
                     "start_location" : {
                        "lat" : -33.8828645,
                        "lng" : 151.208348
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.9 km",
                        "value" : 876
                     },
                     "duration" : {
                        "text" : "3 mins",
                        "value" : 152
                     },
                     "end_location" : {
                        "lat" : -33.8839614,
                        "lng" : 151.2175457
                     },
                     "html_instructions" : "Turn \u003cb\u003eright\u003c/b\u003e onto \u003cb\u003eAlbion St\u003c/b\u003e",
                     "maneuver" : "turn-right",
                     "polyline" : {
                        "points" : "xoxmEs|{y[?I@K@SBMDO^aAN_@L_@`@cADMr@}BHSp@iBRe@@KLgABWLeA@K@QFc@Fg@@OD]XoC@KBULeATuB@KHu@Hm@BUPyATuBF_@Do@?GGaA?CAE?A?EAI"
                     },
                     "start_location" : {
                        "lat" : -33.8817274,
                        "lng" : 151.2085805
                     },
                     "travel_mode" : "DRIVING"
                  },
                  {
                     "distance" : {
                        "text" : "0.1 km",
                        "value" : 123
                     },
                     "duration" : {
                        "text" : "1 min",
                        "value" : 30
                     },
                     "end_location" : {
                        "lat" : -33.8849029,
                        "lng" : 151.21814
                     },
                     "html_instructions" : "Turn \u003cb\u003eright\u003c/b\u003e onto \u003cb\u003eFlinders St\u003c/b\u003e\u003cdiv style=\"font-size:0.9em\"\u003eDestination will be on the left\u003c/div\u003e",
                     "maneuver" : "turn-right",
                     "polyline" : {
                        "points" : "v}xmEut}y[?IACBAFAPEx@_@BAb@SZQ`@U"
                     },
                     "start_location" : {
                        "lat" : -33.8839614,
                        "lng" : 151.2175457
                     },
                     "travel_mode" : "DRIVING"
                  }
               ],
               "via_waypoint" : []
            }
         ],
         "overview_polyline" : {
            "points" : "joumEsmyy[UrAQv@CLHBNBLJFD@A@C@ADCL@JQHKbAm@pBaAbB}@|@{@R_@\\w@VgAFg@?y@BMHOHEn@CdC?v@HNUVa@`@a@\\UTIz@KlDLb@Bb@FFEH@DD@B?@PBfADb@?d@AbBAz@KhB[f@MfBo@rAm@xD{AvBeARMJMZq@?E?GFOLKNCNDHL@Bf@c@lDwAh@Uf@S`DaAjAa@^ElBo@q@{B}@uCI[o@sBEQDU\\wCjCl@pA_D@WHWLq@L_@h@wAvDmIiC[wAQ?IB_@H]~AeEtBiGTq@P_BXgCrAcMXeCt@_HKeBAMBAXG|@a@`B{@"
         },
         "summary" : "Darling Dr",
         "warnings" : [],
         "waypoint_order" : []
      }
   ],
   "status" : "OK"
}
```

The Google Directions API is a service that calculates directions between locations using an HTTP request. Directions may specify origins, destinations and waypoints either as text strings (e.g. "Chicago, IL" or "Darwin, NT, Australia") or as latitude/longitude coordinates. The Directions API can return multi-part directions using a series of waypoints.


## 実装, 解決



## 参考

- https://developers.google.com/maps/documentation/directions/
