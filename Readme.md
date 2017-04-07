*이 프로젝트 소스를 사용할 때 assets 폴더 아래 다운받을 파일을 포함시키면 됩니다*

#1. AsyncTask

##1.1 개념

 AsyncTask는 Thread나 Message 등을 사용하지 않고도 하나의 메인 클래스에서 UI작업과 background 작업을 할 수 있도록 합니다.


![](http://cfile23.uf.tistory.com/image/2420B240577D4A720F8136)

(1) aTask.execute() : thread.start()와 같이 AsyncTask에서는 execute()로 실행합니다.

(2) onPreExcetued() : AsyncTask로 작업을 시작하기 전에 실행되는 메소드입니다.

(3) doInBackground() : background 작업을 진행하는 곳입니다. 

(4) publishProgress() : donInBackground()에서 UI를 업데이트하려고 할 때, 호출되는 곳입니다.

(5) onProgressUpdate() : publcishProgress()가 호출될 때마다 자동으로 호출됩니다.

(6) onPostExcuted() : background 작업이 끝난 후, AsynTask의 결과 파라미터가 전달됩니다. 


##1.2 출력화면

![](http://i.imgur.com/Nfj8RgD.png)

START 버튼을 누르면 앱에 저장되어있는 .mp4파일을 다운 받으면서 SeekBar에 실시간으로 다운로드 실행이 표시됩니다. DELETE 버튼을 누르면 다운받은 파일이 삭제됩니다.

![](http://i.imgur.com/HW7wUJW.png)

환경설정에서 애플리케이션 정보를 확인하면 내장된 데이터가 증가한것을 확인할 수 있습니다.

##1.3 코드

##1.3.1 비디오 파일 앱 위치에 놓기

![](http://i.imgur.com/DOI3zWN.png)

[main] 폴더 하위에 사용할 동영상 파일을 저장합니다. 이 때, 반드시 [assets]라는 폴더명 하위에 두어야 합니다.

##1.3.2 AsyncTask 동작시키기

![](http://i.imgur.com/zm6Z81E.png)

AsyncTask 인스턴스에 파일명을 담아 execute()로 실행시킵니다.

##1.3.3 AsyncTask 내부 클래스

 			public class TestAsync extends AsyncTask<String, Integer, Boolean>

AsyncTask 클래스를 상속해야 합니다. 또한 3개의 인자타입을 받습니다. 왼쪽부터 doInBackground(), onProgressUpdate(), onPostExecute()의 인자로 각각 들어가게 됩니다.

(1) onPreExecute()

![](http://i.imgur.com/yXm99AX.png)

AsyncTask의 background 프로세스 전에 호출되는 함수입니다. 

					AssetManager manager = getAssets(); // 파일 사이즈를 입력받기 위한 함수

					InputStream is = manager.open("Dokkabi.mp4"); // 파일 사이즈를 가져오기 위해 파일 스트림 생성

					int fileSize = is.available(); // stream에 연결된 파일사이즈를 리턴해준다.

					progressBar.setMax(fileSize); // 프로그래스바의 최대값에 파일사이즈 입력

(2) doInBackground()

![](http://i.imgur.com/Cqg7SVw.png)

파일명을 가져와서 assetToDisk()로 전달합니다. 여기서는 파일 다운로드가 진행됩니다.

(3) onPostExecute()

![](http://i.imgur.com/Jvpv94C.png)

doInBackground 가 종료된 후에 호출되는 함수로써, doInBackground의 return 값을 받습니다. 다운로드가 완료되었을 경우, 텍스트뷰에 "완료되었습니다" 를 출력시킵니다.

   

(4) onProgressUpdate()

![](http://i.imgur.com/31A98OY.png)

main thread에서 실행되는 함수입니다. 넘어온 파일길이를 텍스트뷰와 SeekBark에 출력합니다.

(5) assetToDisk()

AsyncTask에서 제공하는게 아니라 사용자가 직접 만든 함수로써 assets에 있는 파일을 쓰기가능한 internal Storage로 복사합니다.

![](http://i.imgur.com/lljD0c9.png)
![](http://i.imgur.com/6eBNP8E.png)

					AssetManager manager = getAssets(); // 1. assets에 있는 파일을 filename으로 읽어온다.

					is = manager.open(filename); // 2. 파일 스트림 생성
					bis = new BufferedInputStream(is); // 3. 버퍼 스트림으로 래핑 ( 한번에 여러개의 데이터를 가져오기 위한 래핑)

					String targetFile = getFullPath(filename); // 4. 저장할 위치에 파일이 없으면 생성
               		File file = new File(targetFile);
              		if (!file.exists()) {
             	       file.createNewFile();
             		}

					fos = new FileOutputStream(file);  // 5. 쓰기 스트림을 생성
					bos = new BufferedOutputStream(fos); // 6. 버퍼 스트림으로 동시에 여러 개의 데이터를 쓰기위한 래핑 (래퍼 클래스???)
	
	                while ((read = bis.read(buffer, 0, 1024)) != -1) { // 읽어올 데이터가 없을때까지 반복문을 돌면서 읽고 쓴다.
	                    bos.write(buffer, 0, read);
	                    publishProgress(read); // AsyncTask의 onProgressUpdate()로 저장된 데이터 용량을 보냅니다.
	                }


##1.3.4 파일명의 전체 디렉토리 구조 가져오기

  		 private String getFullPath(String filename){        
   			     return getFilesDir().getAbsolutePath() + File.separator + filename;
  		  }