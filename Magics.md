# Magics

Magics - библиотека для клиент-серверного взаимодейтсвия с АПИ сервисами, выдающими ответ в формате json.

Библиотека очень простая в использовании и состоит из трех компонентов:

1. `API`
2. `Interactor`
3. `Model`

Начнем с компонента `API`.

## MagicsAPI

Класс API содержит в себе всю логику, единую для всех запросов к конкретному API: как (и нужно ли) вставлять в запрос token, выдает ли API ошибку дл язапроса, путь базового url. Так же в нем можно определить логику для случаев устаревания token'ов.
Класс API для вашего приложения должен наследоваться от MagicsAPI, и в самом минимальном виде должен переопредлить свойство `baseUrl`:

~~~swift
class TetrapakAPI: MagicsAPI {
    override var baseURL: String { return "https://base.com/api/" }
}
~~~

Если для запросов нужен token, то логика его добавления и хранения реализуется так же на уровне API:

~~~swift
class TetrapakAPI: MagicsAPI {
    
    static let shared = TetrapakAPI()
    
    private let kTokenKey = "TetrapakAPI.kTokenKey"
    private let keychain = KeychainSwift()
    
    var token: String? {
        get { return keychain.get(kTokenKey) }
        set {
            if let token = newValue { keychain.set(token, forKey: kTokenKey) }
            else { keychain.delete(kTokenKey) }
        }
    }
    
    override var baseURL: String { return "https://base.com/api/" }
    
    override func modify<T>(request: URLRequest, interactor: T) -> URLRequest where T : MagicsInteractor {
        var request = request
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        if let token = token {
            request.addValue(token, forHTTPHeaderField: "X-Auth-Token")
        }
        return request
    }
}
~~~
Обычно для всех запросов API существует единая логика сообщения о том, что что-то пошло не так. Мы реализуем эту логику в методе `hasErrorFor`:

~~~swift    
    override func hasErrorFor<T>(json: MagicsJSON?, response: URLResponse?, error: Error?, interactor: T) -> Error? where T : MagicsInteractor {        
        guard let json = json else {
            return ResponseError(code: 101, message: "С сервера пришел пустой ответ. Повторите запрос позже")
        }
        
        if json["result"].string == "success" { return nil }
        
        guard let code = json["errors"]?[0]?["code"]?.int, let message = json["errors"]?[0]?["message"]?.string else {
            return ResponseError(code: 100, message: "Произошла неизвестная ошибка. Повторите запрос позже")
        }
        return ResponseError(code: code, message: message)
    }
~~~
В этом методе, если сервер вернул ошибку, то мы создаем и возвращаем объект ошибки (`ResponseError`). Если запрос прошел успешно, то мы возвращаем `nil`. В примере выше мы описали логику, при которой запрос считается успешным, если json, полученный с сервера, не пустой и в поле `result` содержит "success". В противном случае мы создаем и возвращаем ошибку.


## MagicsInteractor

Интерактор – это объект, который отвечает за взаимодействие с конкретным методом API: описывает относительный url, описывает какой запрос нам нужно послать и как обработать ответ от сервера. Чаще всего в приложении есть один класс, реализующий MagicsAPI, и много классов Interactor'ов.


В самом базовом виде ваш интерактор должен только определить свойство относительного url'а (от базового, прописанного в API): 
~~~swift
final class LogoutInteractor: NSObject, MagicsInteractor {
    var relativeURL: String { return "user/logout" }
}
~~~
Как в случае выше, вам может быть необходимо только отправить запрос по указанному url, и никаких дополнительных действий делать не нужно. 

### Изменение запроса
Однако в большинстве случаев вам понадобится изменять запрос, и для этого вам следует переопределить метод `modify(request:`:

~~~swift
final class EditPasswordInteractor: NSObject, MagicsInteractor {
    var method: MagicsMethod { return .post } // по умолчанию выполняется get запрос, но вы можете изменить это, указав здесь нужный тип запроса
    var relativeURL: String { return "changePassword" }
    
    let newPassword: String
    
    override init() {
        fatalError()
    }
    
    init(newPassword: String) {
        self.newPassword = newPassword
    }
    
    func modify(request: URLRequest) -> URLRequest {
        var request = request
        let parameters = [
            "newPassword": newPassword
        ]
        request.setFormDataBody(with: parameters) //добавляем данные в запрос
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        return request
    }
}
~~~
Для добавления данных в запрос обычно используется один из двух методов:
`setFormDataBody(with parameters: [String: String])` – для вставки form даты

`request.setJSONBody(with: [String : Any])` – для вставки данных в формате json
А также ряд методов, для осуществления запроса и его обработки.

### Обработка ответа
Поля, лежащие в корне json'а с сервера, Magics может автоматически распарсить и вставить в поля интерактора. (Это будет описано чуть позже) Для ручной обработки ответа с сервера нужно переопрделеить функцию `process(json: MagicsJSON, response: URLResponse?, api: MagicsAPI)`:

~~~swift
    func process(json: MagicsJSON, response: URLResponse?, api: MagicsAPI){
        testName = json["categories"]["products"][0]["name"].string
    }
~~~
