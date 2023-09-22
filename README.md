# SourceryTemplates

## Mocks:

Based on the standard Sourcery Mocking Template but extended with Mocks for:
- AnyPublisher 
- override annotation 

### AnyPublisher
Example Variable:
```
public protocol AnyPublisherService: AutoMockable {
    var isLoading: AnyPublisher<Bool, Never> { get }
}
```
results in:
```
public class AuthenticationServiceMock: AuthenticationService {

    public init() {}

    public var isLoading: AnyPublisher<Bool, Never> {
        return underlyingIsLoading.eraseToAnyPublisher()
    }
    public var underlyingIsLoading = PassthroughSubject<Bool, Never>()
```

### override annotation
I created this anotation to better mock generic types but it can be used for other things too. It simple overrides the datatype for the mocked function parameters. (Currently not suppored on variables it self)

Example Protocol:
```
public protocol HttpClient: AutoMockable {

    // sourcery:overrideReturn = Any
    func request<T: Decodable, Body: Encodable>(_ type: T.Type, // sourcery:override = Any.Type
    
                                                method: HttpMethod,
                                                path: String,
                                                query: [String: Any]?,
                                                body: Body?, // sourcery:override = Any?
                                                options: HttpClientOptions?) async throws -> T}
```

Results in
```
public class HttpClientMock: HttpClient {
    //MARK: - request<T: Decodable, Body: Encodable>

    ///:Generics not perferctly supported!
    public var requestMethodPathQueryBodyOptionsThrowableError: Error?
    public var requestMethodPathQueryBodyOptionsCallsCount = 0
    public var requestMethodPathQueryBodyOptionsCalled: Bool {
        return requestMethodPathQueryBodyOptionsCallsCount > 0
    }
    public var requestMethodPathQueryBodyOptionsReceivedArguments: (type: Any.Type , method: HttpMethod, path: String, query: [String: Any]?, body: Any? , options: Any? )?
    public var requestMethodPathQueryBodyOptionsReceivedInvocations: [(type: Any.Type , method: HttpMethod, path: String, query: [String: Any]?, body: Any? , options: Any? )] = []

    public var requestMethodPathQueryBodyOptionsReturnValue: Any?
    public var requestMethodPathQueryBodyOptionsClosure: ((Any.Type, HttpMethod, String, [String: Any]?, Any?, Any?) async throws -> Any)?

    public func request<T: Decodable, Body: Encodable>(_ type: T.Type, method: HttpMethod, path: String, query: [String: Any]?, body: Body?, options: HttpClientOptions?) async throws -> T {
        if let error = requestMethodPathQueryBodyOptionsThrowableError {
            throw error
        }
        requestMethodPathQueryBodyOptionsCallsCount += 1
        requestMethodPathQueryBodyOptionsReceivedArguments = (type: type, method: method, path: path, query: query, body: body, options: options)
        requestMethodPathQueryBodyOptionsReceivedInvocations.append((type: type, method: method, path: path, query: query, body: body, options: options))
        if let requestMethodPathQueryBodyOptionsClosure = requestMethodPathQueryBodyOptionsClosure {
            return try await requestMethodPathQueryBodyOptionsClosure(type, method, path, query, body, options) as! T
        } else {
            return requestMethodPathQueryBodyOptionsReturnValue as! T
        }
    }
}
```

