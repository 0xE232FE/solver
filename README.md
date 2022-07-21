# solver
Solving Captchas

Training Data 2443 Samples

```
Usage of solver_linux_amd64
  -host string
        listening Interface (default "127.0.0.1")
  -port string
        listening Port (default "8888")
```

### Response
```
{
    "answer": 3
}
```


```go
// CrynogarSolver solve Captchas like a Ninja
func CrynogarSolver(solverURL string) ogame.CaptchaCallback {
	return func(question, icons []byte) (int64, error) {
		body := &bytes.Buffer{}
		writer := multipart.NewWriter(body)
		part, _ := writer.CreateFormFile("question", "question.png")
		_, _ = io.Copy(part, bytes.NewReader(question))
		part1, _ := writer.CreateFormFile("icons", "icons.png")
		_, _ = io.Copy(part1, bytes.NewReader(icons))
		_ = writer.Close()

		req, _ := http.NewRequest(http.MethodPost, solverURL+"/solve", body)
		req.Header.Add("Content-Type", writer.FormDataContentType())
		resp, _ := http.DefaultClient.Do(req)
		defer resp.Body.Close()
		if resp.StatusCode != 200 {
			by, err := ioutil.ReadAll(resp.Body)
			if err != nil {
				return 0, errors.New("failed to auto solve captcha: " + err.Error())
			}
			return 0, errors.New("failed to auto solve captcha: " + string(by))
		}
		by, _ := ioutil.ReadAll(resp.Body)
		var answerJson struct {
			Answer int64 `json:"answer"`
		}
		if err := json.Unmarshal(by, &answerJson); err != nil {
			return 0, errors.New("failed to auto solve captcha: " + err.Error())
		}
		return answerJson.Answer, nil
	}
}


params := ogame.Params{
    Universe:        universe,
    Username:        username,
    Password:        password,
    Lang:            language,
    AutoLogin:       false,
    CaptchaCallback: CrynogarSolver("http://localhost:8888"),
}
bot, _ := ogame.NewWithParams(params)
err := bot.Login()
```
