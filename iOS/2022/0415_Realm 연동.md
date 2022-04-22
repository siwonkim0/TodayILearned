### Realm의 파일 경로를 알 수 있음

파인더 → 이동 탭에서 이 경로 검색해서 Realm 파일 열어 사용 

```swift
Realm.Configuration.defaultConfiguration.fileURL
```

## trouble shooting

- filter id로 가져오면 안되는 문제
- 똑같은 할일이 여러개 realm에 저장되는 문제
    - primary key 로 id를 지정해서 해결하려함
        - 근데 모델이 변해서 다시 데이터 자체를 삭제하고 재진행해야함
        
        AppDelegate에서 이렇게 해서 삭제함 
        
        ```swift
        autoreleasepool {
            //aa
        }
        try! Realm.deleteFiles(for: Realm.Configuration.defaultConfiguration)
        ```
        
        - primary key 지정하자 같은 할일이 저장되지는 않는데 id가 같으면 에러가 발생함
        
        update 메서드 
        
        primary key가 같으면 자동으로 업데이트해주는 기능이 있음
        
        ```swift
        func update(_ project: Project) {
        //        let memo = realm.object(ofType: ProjectRealm.self, forPrimaryKey: "\(project.id)")
        //
        //        try! realm.write {
        //            memo?.id = project.id
        //            memo?.body = project.body
        //            memo?.title = project.title
        //            memo?.date = project.date
        //            memo?.state = project.state.string
        //        }
            guard let memo = ProjectRealm(project) else {
                return
            }
            try! realm.write {
                realm.add(memo, update: .modified)
            }
        }
        ```
        
- 원격과 로컬 데이터 **동기화** 고민

네트워크에 연결되어있지 않을 때는 로컬 데이터를 보여주는데,
로컬 데이터를 변경하고 나서 네트워크에 연결이 되었을 때 어떻게 처리할지 고민

현재는 레포지토리에서 
네트워크 연결시 원격에서 데이터 가져와서 로컬에 연동시키는건데 
이러면 로컬 변경사항이 다 날라감 

```swift
func bindProjects() -> BehaviorRelay<[Project]> {
    if networkConnection.isConnected == true {
        remoteDataSource.fetch()
            .subscribe(onSuccess: { fetchedProjects in
            self.projects.accept(fetchedProjects)
                fetchedProjects.forEach { project in
                    self.localDataSource.syncronize(with: project)
                }
        }).disposed(by: disposeBag)
    } else {
        localDataSource.fetch()
            .subscribe(onSuccess: { fetchedProjects in
            self.projects.accept(fetchedProjects)
        }).disposed(by: disposeBag)
    }
    return projects
}
```

그래서 로컬을 기준으로 원격 데이터중 id가 같은 것을 삭제하고 
로컬과 원격 데이터를 합쳐서 반환하는 메서드를 만듦 
그래서 처음 화면이 로딩될때는 이제 잘
네트워크의 유무에 따라 동기화가 잘 되는데... 

네트워크 연결시 로컬 기준으로 원격도 업데이트하고 난 뒤 
업데이트된 원격 데이터를 가져옴 

```swift
func bindProjects() -> BehaviorRelay<[Project]> {
//        networkConnection.isConnected = false
    if networkConnection.isConnected == true {
        self.synchronize()
            .subscribe(onSuccess: { fetchedProjects in
            self.projects.accept(fetchedProjects)
                fetchedProjects.forEach { project in
                    self.remoteDataSource.append(project)
                }
        }).disposed(by: disposeBag)
    } else {
        localDataSource.fetch()
            .subscribe(onSuccess: { fetchedProjects in
            self.projects.accept(fetchedProjects)
        }).disposed(by: disposeBag)
    }
    return projects
}

func synchronize() -> Single<[Project]> {
    let local = localDataSource.fetch()
    let remote = remoteDataSource.fetch()
    // Local을 기준으로 Local과 id가 같은 Remote 삭제 후 Local, Remote 배열 병합
    return Single.zip(local, remote).map { local, remote in
        var remoteCopy = remote
        let updatedLocal = local.map { project -> Project in
            let updatedIndex = remoteCopy.firstIndex(where: { $0.id == project.id })
            
            if let updatedIndex = updatedIndex {
                let updated = remoteCopy[updatedIndex]
                remoteCopy.remove(at: updatedIndex)
                return updated
            } else {
                return project
            }
        }
        return updatedLocal + remoteCopy
    }
}
```

- 음 근데 뭔가 수정된 것만 자동으로 관리해주는 뭔가가 있을까 찾아봤다

[https://stackoverflow.com/questions/54096606/realm-syncing-with-large-collection-on-firestore-architectural-questions-iss](https://stackoverflow.com/questions/54096606/realm-syncing-with-large-collection-on-firestore-architectural-questions-iss)

ios synchronize realm firebase 로 검색함
→ firebase를 기준으로 realm이 바뀐게 있나 보는건가..

뭘 하나를 기준으로 두어야할듯
timestamp 얘기도 하고
flag 얘기도 하는데 다시 내일 보자

[https://www.mongodb.com/community/forums/t/syncing-between-firestore-and-realm/123532](https://www.mongodb.com/community/forums/t/syncing-between-firestore-and-realm/123532)
