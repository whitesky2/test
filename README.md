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
