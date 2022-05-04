---
markmap:
  color: '#1abc9c'
---

# URLSession

## init

- ==shared==
- URLSessionConfiguration
- URLSessionDelegate
- OperationQueue

## URLSessionTask

- ==URLSessionDataTask==
  - async API
    - `func data(from: URL, delegate: URLSessionTaskDelegate?) -> (Data, URLResponse)`
    - `func data(for: URLRequest, delegate: URLSessionTaskDelegate?) -> (Data, URLResponse)`
    - bytes*
      - `func bytes(from: URL, delegate: URLSessionTaskDelegate?) -> (URLSession.AsyncBytes, URLResponse)`
      - `func bytes(for: URLRequest, delegate: URLSessionTaskDelegate?) -> (URLSession.AsyncBytes, URLResponse)`
  - completion handler API
    - `func dataTask(with: URL, completionHandler: (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask`
    - `func dataTask(with: URLRequest, completionHandler: (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask`
  - combine API
    - `func dataTaskPublisher(for: URL) -> URLSession.DataTaskPublisher`
    - `func dataTaskPublisher(for: URLRequest) -> URLSession.DataTaskPublisher`
  - normal API
    - `func dataTask(with: URL) -> URLSessionDataTask`
    - `func dataTask(with: URLRequest) -> URLSessionDataTask`
- URLSessionUploadTask
  - async API
    - `func upload(for: URLRequest, from: Data, delegate: URLSessionTaskDelegate?) -> (Data, URLResponse)`
    - `func upload(for: URLRequest, fromFile: URL, delegate: URLSessionTaskDelegate?) -> (Data, URLResponse)`
  - completion handler API
    - `func uploadTask(with: URLRequest, from: Data?, completionHandler: (Data?, URLResponse?, Error?) -> Void) -> URLSessionUploadTask`
    - `func uploadTask(with: URLRequest, fromFile: URL, completionHandler: (Data?, URLResponse?, Error?) -> Void) -> URLSessionUploadTask`
  - normal API
    - `func uploadTask(with: URLRequest, from: Data) -> URLSessionUploadTask`
    - `func uploadTask(with: URLRequest, fromFile: URL) -> URLSessionUploadTask`
    - `func uploadTask(withStreamedRequest: URLRequest) -> URLSessionUploadTask`
- URLSessionDownloadTask
  - async API
    - `func download(from: URL, delegate: URLSessionTaskDelegate?) -> (URL, URLResponse)`
    - `func download(for: URLRequest, delegate: URLSessionTaskDelegate?) -> (URL, URLResponse)`
    - `func download(resumeFrom: Data, delegate: URLSessionTaskDelegate?) -> (URL, URLResponse)`
  - completion handler API
    - `func downloadTask(with: URL, completionHandler: (URL?, URLResponse?, Error?) -> Void) -> URLSessionDownloadTask`
    - `func downloadTask(with: URLRequest, completionHandler: (URL?, URLResponse?, Error?) -> Void) -> URLSessionDownloadTask`
    - `func downloadTask(withResumeData: Data, completionHandler: (URL?, URLResponse?, Error?) -> Void) -> URLSessionDownloadTask`
  - normal API
    - `func downloadTask(with: URL) -> URLSessionDownloadTask`
    - `func downloadTask(with: URLRequest) -> URLSessionDownloadTask`
    - `func downloadTask(withResumeData: Data) -> URLSessionDownloadTask`
- URLSessionStreamTask
  - normal API
    - `func streamTask(withHostName: String, port: Int) -> URLSessionStreamTask`
- URLSessionWebSocketTask
  - normal API
    - `func webSocketTask(with: URL) -> URLSessionWebSocketTask`
    - `func webSocketTask(with: URLRequest) -> URLSessionWebSocketTask`
    - `func webSocketTask(with: URL, protocols: [String]) -> URLSessionWebSocketTask`

## managing

- `func finishTasksAndInvalidate()`
- `func flush(completionHandler: () -> Void)`
- `func getAllTasks(completionHandler: ([URLSessionTask]) -> Void)`
- `func invalidateAndCancel()`
- `func reset(completionHandler: () -> Void)`
- `var sessionDescription: String?`
- 
   ```
    func getTasksWithCompletionHandler(
    ([URLSessionDataTask], [URLSessionUploadTask], [URLSessionDownloadTask]) -> Void)
   ```
