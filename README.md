# 티웨이브 면접자 윤시완
안녕하세요, 8월 11일 면접보았던 윤시완입니다
코딩 스타일을 확인하시기 위해 요청하셨던 소스 코드 일부를 공유해 드립니다.

```swift
 func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
        let localName = advertisementData["kCBAdvDataLocalName"] as? String
        
        if let name = localName {      // 'O2OXCF2715'라는 이름을 받는다면,
            if(name.hasPrefix("O2OX")) {
                let end = name.index(name.endIndex, offsetBy: -4)
                
                guard let result = Int(name[end..<name.endIndex], radix: 16) else {
                    return
                }
                self.yid = result
                
                if let target = self.target_yid, self.yid != target {
                    return
                }
                
                LoadCabinetData()
                
                timer?.invalidate()
                central.stopScan()
            }
        }
    }
```
위 소스는 블루투스를 통해 Scan 하여 보관함 id를 가지고 오는 로직입니다.

```swift
let headers: HTTPHeaders = [
                "Accept": "application/json",
                "Authorization": "Bearer \(self.serverHost.token)"
            ]
            
Alamofire.request("\(serverHost.serverUrl)v2/Locker", method: .get, headers: headers).responseJSON {
    response in
    if response.error != nil {
        DialogViewController.showDialog(view: self, header: "에러", body_message: "네트워크 시도 중 실패했습니다.", handler: nil)
        return
    }

    if response.response?.statusCode == 200 || response.response?.statusCode == 304 {
        if let data = response.data, let recvData = try? JSONSerialization.jsonObject(with: data, options: []) as? [String: Any], let converted = try? JSONSerialization.data(withJSONObject: recvData?["data"] ?? "", options: []), let mArr = try? decoder.decode([Locker].self, from: converted) {
            print("data check: \(mArr.count)")
            self.receivedCab.append(contentsOf: mArr)

            DispatchQueue.main.async {
                self.cabinetList.beginUpdates()
                self.cabinetList.reloadSections(IndexSet(integer: 0), with: .automatic)
                self.cabinetList.endUpdates()
                self.refresh.endRefreshing()
            }
        } else {
            DialogViewController.showDialog(view: self, header: "내부 에러", body_message: "내부 에러로 인해 데이터를 가져올 수 없었습니다.", handler: nil)
        }
    } else {
        DialogViewController.showResult(view: self, statusCode: response.response?.statusCode ?? 600, handler: nil)
    }
}
```

위 소스는 Alamofire를 이용하여 restful Api를 call해서 데이터를 가지고 오는 부분을 일부 공유해 드립니다.

```swift
do {
     let ori_data=try JSONSerialization.jsonObject(with: data, options: []) as? [String:AnyObject]   // 원 데이터
     var columnIndex: Int = -1   // 그릴 열의 번호가 바뀌는 것을 포착하기 위해 사용
     var sumOfWidth: Double = 0  // 전체 보관함의 너비 값(scroll view의 너비를 지정하기 위해)
     var firstMyThing: Int = -1   // 보관함에서 가장 처음으로 나오는 내 물건의 x 값
     var countMyThing: Bool = false   // 실제 내 물건이 있는지 체크함.

     if let j_data = ori_data, let temp = j_data["data"] as? [String:AnyObject] {
         arr_data = HSCApplebox(data: temp, width: Int(self.bounds.width), height: Int(self.bounds.height)).calc()

         for i in 0..<arr_data.count {
             let btn=UIButton(frame: CGRect(x: (arr_data[i].location?.left!)!+4, y: (arr_data[i].location?.top!)!+4, width: (arr_data[i].location?.getWidth())!-4, height: (arr_data[i].location?.getHeight())!-4))
             btn.tag=i
             btn.setTitle(arr_data[i].box?.label, for: .normal)
             btn.titleLabel?.font = UIFont.systemFont(ofSize: 25.0)
             btn.setTitleColor(UIColor.black, for: .normal)
             btn.layer.borderWidth=1
             btn.layer.borderColor=UIColor(red: 0.8235, green: 0.8235, blue: 0.8235, alpha: 1).cgColor
             btn.titleLabel?.font=UIFont(name: "NotoSansCJKkr-Bold", size: 12.0)
             btn.addTarget(self, action: #selector(clickMe(sender:)), for: .touchUpInside)

             if(columnIndex != arr_data[i].box?.col) // 열이 바뀌었을 때 sumOfWidth의 값을 늘려서 그릴 영역을 넓힘. 그리고 라벨링을 붙임.
             {   
                 columnIndex=(arr_data[i].box?.col)!
                 sumOfWidth=sumOfWidth + (arr_data[i].location?.getWidth())!
             }

             let colorName=arr_data[i].box?.status
             if(self.mode == .getBack) {
                 if((arr_data[i].box?.toHp == self.serverHost.myInfo.hp) && ((arr_data[i].box?.status == "A"))) {   // 내물건이라면, 초록색으로 표시한다.
                     if(firstMyThing == -1) {
                         btn.backgroundColor = self.clickedColor
                         firstMyThing=Int(btn.frame.origin.x)
                         countMyThing=true
                         selectedCabinet=btn
                     } else {
                         btn.backgroundColor = self.normalColor
                     }
                 } else {    // 이 보관함이 내것이 아니라면 적색으로 표시하고 터치할 수 없도록 한다.
                     btn.backgroundColor = self.restrictColor
                     btn.isEnabled=false
                 }
             } else {
                 if(colorName=="B") {    // 현재 이 캐비닛이 사용할 수 있는 캐비닛이라면, 흰색으로 표시.
                     btn.backgroundColor = self.normalColor
                 } else {    // 그렇지 않다면 적색으로 표시하고 터치할 수 없도록 만듦.
                     btn.backgroundColor = self.restrictColor
                     btn.isEnabled=false
                 }
             }

             if(arr_data[i].box?.label!=="MAIN") {
                 btn.backgroundColor = self.mainColor
                 btn.setTitle("", for: .normal)
                 btn.setImage(UIImage(named: "imgKioskScreen"), for: .normal)
                 btn.contentEdgeInsets = UIEdgeInsets(top: 5.0, left: 5.0, bottom: 5.0, right: 5.0)
             }

             self.addSubview(btn)
         }
         self.contentSize=CGSize(width: CGFloat(sumOfWidth), height: self.bounds.height)

         // 가장 처음 좌측에서 등장하는 내물건 위치로 스크롤링한다.
         let frame=CGRect(x: firstMyThing, y: 0, width: Int(self.bounds.width), height: Int(self.bounds.height))
         self.scrollRectToVisible(frame, animated: true)

         if(mode==Mode.getBack && countMyThing==false) {     // 실제 내 물건이 없을 때
             DialogViewController.showDialog(view: self.superVC!, header: "물건 없음", body_message: "보관함에 물건이 없습니다.", handler: {
                 (UIAlertAction) -> Void in

                 self.superVC!.navigationController?.popToRootViewController(animated: true)
             })
         }
     } else {
         self.errorProcessing(header: "internal Error", body: "내부 에러")
     }
 } catch {
     DialogViewController.showDialog(view: self.superVC!, header: "Error", body_message: "parse error", handler: nil)
 }
```

위 소스는 앱에서 택배보관함의 데이터를 가지고와서 앱에 그려주는 소스 로직입니다.
