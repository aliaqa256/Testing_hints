# Testing_hints

#### HIGHER ORDER FUNCTION : PASS TYPE FUNCTION  TO THE TESTED FUNCTION -> AT TESTING TIME YOU CAN PASS THE FUNCTION YOU NEED 
```
// main package
type Sqlopener func() (*sql.DB, error)
func OpenDB(sqlopener Sqlopener) error {
    db, err := sqlopener()
    if err != nil {
        return err
    }
    defer db.Close()
    // do something with db
}
// test package
func TestOpenDB(t *testing.T) {
    err := OpenDB(func() (*sql.DB, error) {
        return &sql.DB{}, nil
    })
    if err != nil {
        t.Error(err)
    }
}
```


-----------------------------------------------------------------------------------------------------------------------------
#### MOMKEY PATCHING :  DEFINE A GLOBAL VARIABLE IN THE MAIN PACKAGE AND THEN IN THE TEST PACKAGE YOU CAN OVERWRITE THE VALUE OF THE GLOBAL VARIABLE
```
// main package
var GetTodayWeek = GetTodayWeekdayInUTC
func IsInTimeRange(val string) bool {
	todayWeekday := GetTodayWeek()
}
// test package
func TestIsInTimeRange(t *testing.T) {
    GetTodayWeek = func() time.Weekday {
        return time.Monday
    }
}
```
-----------------------------------------------------------------------------------------------------------------------------
#### USE INTERFACE AND EMBEDDING TO MOCK THE DEPENDENCIES : CREATE INTERFACE FOR ARGUMENTS AND RETURN VALUES OF THE FUNCTION YOU WANT TO TEST AND THEN MOCK THE INTERFACE IN THE TEST PACKAGE WITH IMPLEMENTATION YOU WANT
```
// main package
type TimeProvider interface {
    Now() time.Time
}
type TimeRange struct {
    Start time.Time
    End   time.Time
}
func IsInTimeRange(val string, tp TimeProvider) bool {
    now := tp.Now()
}
// test package
type MockTimeProvider struct {
    NowFunc func() time.Time
}
func (mtp *MockTimeProvider) Now() time.Time {
    return mtp.NowFunc()
}
func TestIsInTimeRange(t *testing.T) {
    mtp := &MockTimeProvider{
        NowFunc: func() time.Time {
            return time.Date(2019, 1, 1, 0, 0, 0, 0, time.UTC)
        },
    }
    if !IsInTimeRange("10:00-12:00", mtp) {
        t.Error("expected to be in time range")
    }
}
```
-----------------------------------------------------------------------------------------------------------------------------
#### PASS THE DYANMIC VALUE TO THE FUNCTION YOU WANT TO TEST : PASS THE VALUE YOU WANT TO TEST TO THE FUNCTION YOU WANT TO TEST
```
// main package
func HTTPCall(url string) (string, error) {
    resp, err := http.Get(url)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return "", err
    }
    return string(body), nil
}
// test package
func TestHTTPCall(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello, client")
    }))
    defer ts.Close()
    body, err := HTTPCall(ts.URL)
    if err != nil {
        t.Error(err)
    }
    if body != "Hello, client" {
        t.Errorf("expected body to be %q, got %q", "Hello, client", body)
    }
}
```
