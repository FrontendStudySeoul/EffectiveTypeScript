이펙티브 타입스크립트를 읽어보면서 타입스크립트가 어떻게 동작하는지 간략하더라도 알아두면 좋을 것 같아서 정리해봤습니다. 타입스크립트의 동작을 이해하고 책을 공부한다면 이해하기 쉬워질 것 같습니다.

## 타입스크립트 컴파일러 구조

타입스크립트가 자바스크립트로 컴파일 되기 위해서 컴파일러가 코드를 해석하고 변환하는 과정이 필요합니다. 이 과정을 크게 5단계로 나눌 수 있습니다.  [Typescript Code](https://github.com/Microsoft/TypeScript/tree/main/src/compiler)

1. **Scanner**
2. **Parser**
3. **Binder**
4. **Checker**
5. **Emiiter**

흐름을 간단히 요약하자면 `SourceCode → **Scanner** → Token Stream → **Parser →** AST → **Binder** → Symbol table`, `AST + Symbol table -> **Checker** -> Type Validation`, `AST + Checker → **Emitter** → JS` Scanner와 Parser, Binder가 타입스크립트 컴파일에 필요한 재료를 만들어주는 역할을 합니다. 이렇게 만들어진 AST와 Symbol table을 가지고 Checker와 Emiiter를 실행합니다. Checker에서는 타입과 문법에 대한 검사를, Emiiter는 검증된 AST를 JS로 재구성해주는 역할을 합니다. 

### Scanner

```tsx
function createScanner(languageVersion: ScriptTarget,
    skipTrivia: boolean,
    languageVariant = LanguageVariant.Standard,
    textInitial?: string,
    onError?: ErrorCallback,
    start?: number,
    length?: number): Scanner
```

scanner 단계에서는 타입스크립트의 소스 코드를 분석하는 단계입니다. scanner가 코드를 읽으며(scan) 코드를 의미 있는 가장 작은 단위인 토큰(SyntaxKind)으로 분할합니다. 즉, 토큰화(Tokenizing) 하는 과정이라고 보면 됩니다.

```tsx
const foo = "string";
// 이 코드는 const, foo, =, "string", ;으로 분할됩니다.

{
	kind: 84 // SyntaxKind.ConstKeyword
},
{
	kind: 80, // SyntaxKind.Identifier
	escapedText: "foo"
},
{
	kind: 64 // SyntaxKind.EqualsToken
},
{
	kind: 11 // SyntaxKind.StringLiteral
	textSourceNode : {
		// 여긴 Identifier외에도 다양한 것들이 들어갈 수 있습니다.
		kind: 80, // SyntaxKind.Identifier
		escapedText: "string"
	},
	singleQuote: false
},
{
	kind: 27 // SyntaxKind.SemicolonToken
}
```
<details>
<summary>토큰(SyntaxKind) 종류</summary>
<pre>
 export const textToKeywordObj: MapLike<KeywordSyntaxKind> = {
        abstract: SyntaxKind.AbstractKeyword,
        accessor: SyntaxKind.AccessorKeyword,
        any: SyntaxKind.AnyKeyword,
        as: SyntaxKind.AsKeyword,
        asserts: SyntaxKind.AssertsKeyword,
        assert: SyntaxKind.AssertKeyword,
        bigint: SyntaxKind.BigIntKeyword,
        boolean: SyntaxKind.BooleanKeyword,
        break: SyntaxKind.BreakKeyword,
        case: SyntaxKind.CaseKeyword,
        catch: SyntaxKind.CatchKeyword,
        class: SyntaxKind.ClassKeyword,
        continue: SyntaxKind.ContinueKeyword,
        const: SyntaxKind.ConstKeyword,
        ["" + "constructor"]: SyntaxKind.ConstructorKeyword,
        debugger: SyntaxKind.DebuggerKeyword,
        declare: SyntaxKind.DeclareKeyword,
        default: SyntaxKind.DefaultKeyword,
        delete: SyntaxKind.DeleteKeyword,
        do: SyntaxKind.DoKeyword,
        else: SyntaxKind.ElseKeyword,
        enum: SyntaxKind.EnumKeyword,
        export: SyntaxKind.ExportKeyword,
        extends: SyntaxKind.ExtendsKeyword,
        false: SyntaxKind.FalseKeyword,
        finally: SyntaxKind.FinallyKeyword,
        for: SyntaxKind.ForKeyword,
        from: SyntaxKind.FromKeyword,
        function: SyntaxKind.FunctionKeyword,
        get: SyntaxKind.GetKeyword,
        if: SyntaxKind.IfKeyword,
        implements: SyntaxKind.ImplementsKeyword,
        import: SyntaxKind.ImportKeyword,
        in: SyntaxKind.InKeyword,
        infer: SyntaxKind.InferKeyword,
        instanceof: SyntaxKind.InstanceOfKeyword,
        interface: SyntaxKind.InterfaceKeyword,
        intrinsic: SyntaxKind.IntrinsicKeyword,
        is: SyntaxKind.IsKeyword,
        keyof: SyntaxKind.KeyOfKeyword,
        let: SyntaxKind.LetKeyword,
        module: SyntaxKind.ModuleKeyword,
        namespace: SyntaxKind.NamespaceKeyword,
        never: SyntaxKind.NeverKeyword,
        new: SyntaxKind.NewKeyword,
        null: SyntaxKind.NullKeyword,
        number: SyntaxKind.NumberKeyword,
        object: SyntaxKind.ObjectKeyword,
        package: SyntaxKind.PackageKeyword,
        private: SyntaxKind.PrivateKeyword,
        protected: SyntaxKind.ProtectedKeyword,
        public: SyntaxKind.PublicKeyword,
        override: SyntaxKind.OverrideKeyword,
        out: SyntaxKind.OutKeyword,
        readonly: SyntaxKind.ReadonlyKeyword,
        require: SyntaxKind.RequireKeyword,
        global: SyntaxKind.GlobalKeyword,
        return: SyntaxKind.ReturnKeyword,
        satisfies: SyntaxKind.SatisfiesKeyword,
        set: SyntaxKind.SetKeyword,
        static: SyntaxKind.StaticKeyword,
        string: SyntaxKind.StringKeyword,
        super: SyntaxKind.SuperKeyword,
        switch: SyntaxKind.SwitchKeyword,
        symbol: SyntaxKind.SymbolKeyword,
        this: SyntaxKind.ThisKeyword,
        throw: SyntaxKind.ThrowKeyword,
        true: SyntaxKind.TrueKeyword,
        try: SyntaxKind.TryKeyword,
        type: SyntaxKind.TypeKeyword,
        typeof: SyntaxKind.TypeOfKeyword,
        undefined: SyntaxKind.UndefinedKeyword,
        unique: SyntaxKind.UniqueKeyword,
        unknown: SyntaxKind.UnknownKeyword,
        var: SyntaxKind.VarKeyword,
        void: SyntaxKind.VoidKeyword,
        while: SyntaxKind.WhileKeyword,
        with: SyntaxKind.WithKeyword,
        yield: SyntaxKind.YieldKeyword,
        async: SyntaxKind.AsyncKeyword,
        await: SyntaxKind.AwaitKeyword,
        of: SyntaxKind.OfKeyword,
    };
    
    const textToKeyword = new Map(Object.entries(textToKeywordObj));
    
    const textToToken = new Map(Object.entries({
        ...textToKeywordObj,
        "{": SyntaxKind.OpenBraceToken,
        "}": SyntaxKind.CloseBraceToken,
        "(": SyntaxKind.OpenParenToken,
        ")": SyntaxKind.CloseParenToken,
        "[": SyntaxKind.OpenBracketToken,
        "]": SyntaxKind.CloseBracketToken,
        ".": SyntaxKind.DotToken,
        "...": SyntaxKind.DotDotDotToken,
        ";": SyntaxKind.SemicolonToken,
        ",": SyntaxKind.CommaToken,
        "<": SyntaxKind.LessThanToken,
        ">": SyntaxKind.GreaterThanToken,
        "<=": SyntaxKind.LessThanEqualsToken,
        ">=": SyntaxKind.GreaterThanEqualsToken,
        "==": SyntaxKind.EqualsEqualsToken,
        "!=": SyntaxKind.ExclamationEqualsToken,
        "===": SyntaxKind.EqualsEqualsEqualsToken,
        "!==": SyntaxKind.ExclamationEqualsEqualsToken,
        "=>": SyntaxKind.EqualsGreaterThanToken,
        "+": SyntaxKind.PlusToken,
        "-": SyntaxKind.MinusToken,
        "**": SyntaxKind.AsteriskAsteriskToken,
        "*": SyntaxKind.AsteriskToken,
        "/": SyntaxKind.SlashToken,
        "%": SyntaxKind.PercentToken,
        "++": SyntaxKind.PlusPlusToken,
        "--": SyntaxKind.MinusMinusToken,
        "<<": SyntaxKind.LessThanLessThanToken,
        "</": SyntaxKind.LessThanSlashToken,
        ">>": SyntaxKind.GreaterThanGreaterThanToken,
        ">>>": SyntaxKind.GreaterThanGreaterThanGreaterThanToken,
        "&": SyntaxKind.AmpersandToken,
        "|": SyntaxKind.BarToken,
        "^": SyntaxKind.CaretToken,
        "!": SyntaxKind.ExclamationToken,
        "~": SyntaxKind.TildeToken,
        "&&": SyntaxKind.AmpersandAmpersandToken,
        "||": SyntaxKind.BarBarToken,
        "?": SyntaxKind.QuestionToken,
        "??": SyntaxKind.QuestionQuestionToken,
        "?.": SyntaxKind.QuestionDotToken,
        ":": SyntaxKind.ColonToken,
        "=": SyntaxKind.EqualsToken,
        "+=": SyntaxKind.PlusEqualsToken,
        "-=": SyntaxKind.MinusEqualsToken,
        "*=": SyntaxKind.AsteriskEqualsToken,
        "**=": SyntaxKind.AsteriskAsteriskEqualsToken,
        "/=": SyntaxKind.SlashEqualsToken,
        "%=": SyntaxKind.PercentEqualsToken,
        "<<=": SyntaxKind.LessThanLessThanEqualsToken,
        ">>=": SyntaxKind.GreaterThanGreaterThanEqualsToken,
        ">>>=": SyntaxKind.GreaterThanGreaterThanGreaterThanEqualsToken,
        "&=": SyntaxKind.AmpersandEqualsToken,
        "|=": SyntaxKind.BarEqualsToken,
        "^=": SyntaxKind.CaretEqualsToken,
        "||=": SyntaxKind.BarBarEqualsToken,
        "&&=": SyntaxKind.AmpersandAmpersandEqualsToken,
        "??=": SyntaxKind.QuestionQuestionEqualsToken,
        "@": SyntaxKind.AtToken,
        "#": SyntaxKind.HashToken,
        "`": SyntaxKind.BacktickToken,
    }));
</pre>
</details>


스캐너는 단순히  코드를 분류하고 필요한 정보를 넣는 것 이상으로 토큰의 위치와 토큰의 text, value 등등 많은 것들을 알 수 있도록 정리해주는 역할을 해줍니다.
<details>
<summary>Scanner의 기능들</summary>
<pre>
     var scanner: Scanner = {
            getTokenFullStart: () => fullStartPos,
            getStartPos: () => fullStartPos,
            getTokenEnd: () => pos,
            getTextPos: () => pos,
            getToken: () => token,
            getTokenStart: () => tokenStart,
            getTokenPos: () => tokenStart,
            getTokenText: () => text.substring(tokenStart, pos),
            getTokenValue: () => tokenValue,
            hasUnicodeEscape: () => (tokenFlags & TokenFlags.UnicodeEscape) !== 0,
            hasExtendedUnicodeEscape: () => (tokenFlags & TokenFlags.ExtendedUnicodeEscape) !== 0,
            hasPrecedingLineBreak: () => (tokenFlags & TokenFlags.PrecedingLineBreak) !== 0,
            hasPrecedingJSDocComment: () => (tokenFlags & TokenFlags.PrecedingJSDocComment) !== 0,
            isIdentifier: () => token === SyntaxKind.Identifier || token > SyntaxKind.LastReservedWord,
            isReservedWord: () => token >= SyntaxKind.FirstReservedWord && token <= SyntaxKind.LastReservedWord,
            isUnterminated: () => (tokenFlags & TokenFlags.Unterminated) !== 0,
            getCommentDirectives: () => commentDirectives,
            getNumericLiteralFlags: () => tokenFlags & TokenFlags.NumericLiteralFlags,
            getTokenFlags: () => tokenFlags,
            reScanGreaterToken,
            reScanAsteriskEqualsToken,
            reScanSlashToken,
            reScanTemplateToken,
            reScanTemplateHeadOrNoSubstitutionTemplate,
            scanJsxIdentifier,
            scanJsxAttributeValue,
            reScanJsxAttributeValue,
            reScanJsxToken,
            reScanLessThanToken,
            reScanHashToken,
            reScanQuestionToken,
            reScanInvalidIdentifier,
            scanJsxToken,
            scanJsDocToken,
            scanJSDocCommentTextToken,
            scan,
            getText,
            clearCommentDirectives,
            setText,
            setScriptTarget,
            setLanguageVariant,
            setOnError,
            resetTokenState,
            setTextPos: resetTokenState,
            setInJSDocType,
            tryScan,
            lookAhead,
            scanRange,
        };
</pre>
</details>


정확한 토큰의 생성 시점은 Parser 단계에서 생성이 됩니다. 생성된 토큰은 추상 구문 트리(AST)의 구성 요소인 Node를 나타내는 역할을 하게 됩니다.

### Parser

Parser 단계에서는 Scanner에서 만들어진 토큰을 조합하여 추상 구문 트리(Abstract Syntax Tree)를 생성하는 단계입니다. AST는 소스 코드의 구조를 표현하는 데이터 구조이며 소스 코드의 Syntax 정보를 포함합니다. Syntax 정보를 가지고 있는 Node를 트리 구조 형태로 가지고 있는 것이 AST입니다.

[Typescript AST Viewer](https://ts-ast-viewer.com/#code/MYewdgzgLgBAZiEMC8MBEgJUcKqNgNVcByDgKWNoxA)
![image](https://github.com/FrontendStudySeoul/TypeScript/assets/59603529/75c3a717-0cfd-4b8b-9e8c-a262f472608b)

먼저 Node를 만들기 위한 재료인 토큰을 생성합니다.

```tsx
var scanner = createScanner(ScriptTarget.Latest, /*skipTrivia*/ true);
```

그리고 토큰들을 토대로 AST의 구성 요소인 Node를 만드는 createNodeFactory를 선언합니다.

```tsx
var factory = createNodeFactory(NodeFactoryFlags.NoParenthesizerRules | NodeFactoryFlags.NoNodeConverters | NodeFactoryFlags.NoOriginalNode, baseNodeFactory);

var {
    createNodeArray: factoryCreateNodeArray,
    createNumericLiteral: factoryCreateNumericLiteral,
    createStringLiteral: factoryCreateStringLiteral,
    createLiteralLikeNode: factoryCreateLiteralLikeNode,
    createIdentifier: factoryCreateIdentifier,
    createPrivateIdentifier: factoryCreatePrivateIdentifier,
    createToken: factoryCreateToken,
    createArrayLiteralExpression: factoryCreateArrayLiteralExpression,
    createObjectLiteralExpression: factoryCreateObjectLiteralExpression,
    createPropertyAccessExpression: factoryCreatePropertyAccessExpression,
    createPropertyAccessChain: factoryCreatePropertyAccessChain,
    createElementAccessExpression: factoryCreateElementAccessExpression,
    createElementAccessChain: factoryCreateElementAccessChain,
    createCallExpression: factoryCreateCallExpression,
    createCallChain: factoryCreateCallChain,
    createNewExpression: factoryCreateNewExpression,
    createParenthesizedExpression: factoryCreateParenthesizedExpression,
    createBlock: factoryCreateBlock,
    createVariableStatement: factoryCreateVariableStatement,
    createExpressionStatement: factoryCreateExpressionStatement,
    createIfStatement: factoryCreateIfStatement,
    createWhileStatement: factoryCreateWhileStatement,
    createForStatement: factoryCreateForStatement,
    createForOfStatement: factoryCreateForOfStatement,
    createVariableDeclaration: factoryCreateVariableDeclaration,
    createVariableDeclarationList: factoryCreateVariableDeclarationList,
} = factory;
```

그리고 만들어진 토큰과 factort를 토대로 Node를 생성합니다.

```tsx
// literal Node code
function parseLiteralLikeNode(kind: SyntaxKind): LiteralLikeNode {
    const pos = getNodePos();
    const node =
        isTemplateLiteralKind(kind) ? factory.createTemplateLiteralLikeNode(kind, scanner.getTokenValue(), getTemplateLiteralRawText(kind), scanner.getTokenFlags() & TokenFlags.TemplateLiteralLikeFlags) :
        // Note that theoretically the following condition would hold true literals like 009,
        // which is not octal. But because of how the scanner separates the tokens, we would
        // never get a token like this. Instead, we would get 00 and 9 as two separate tokens.
        // We also do not need to check for negatives because any prefix operator would be part of a
        // parent unary expression.
        kind === SyntaxKind.NumericLiteral ? factoryCreateNumericLiteral(scanner.getTokenValue(), scanner.getNumericLiteralFlags()) :
        kind === SyntaxKind.StringLiteral ? factoryCreateStringLiteral(scanner.getTokenValue(), /*isSingleQuote*/ undefined, scanner.hasExtendedUnicodeEscape()) :
        isLiteralKind(kind) ? factoryCreateLiteralLikeNode(kind, scanner.getTokenValue()) :
        Debug.fail();

    if (scanner.hasExtendedUnicodeEscape()) {
        node.hasExtendedUnicodeEscape = true;
    }

    if (scanner.isUnterminated()) {
        node.isUnterminated = true;
    }

    nextToken();
    return finishNode(node, pos);
}
```

토큰을 토대로 Node를 만드는 과정에서 1차적으로 문법 오류를 확인하는 역할을 합니다. 이 과정에서 문법 규칙에 어긋나는 코드를 발견한다면 Parser는 오류 메세지를 생성해 사용자(개발자)에게 알립니다. 이 단계에서는 아직 타입이 생성되지 않았기 때문에 타입 오류를 잡아주진 않습니다. 

### Binder

Binder 단계는 Parser가 생성한 AST를 기반으로 심볼 테이블을 생성하는 단계입니다. 심볼 테이블은 코드 내에서 정의된 변수, 함수, 클래스 등 모든 식별자들의 정보를 담고 있습니다. 이 정보는 식별자의 유효 범위(scope), 타입, 선언 위치 등을 포함됩니다.

예로 `const foo: string = 'string'` 코드의 경우 `foo`라는 심볼이 어떤 타입을 가지는지, 어디서 선언되는지(scope) 등의 정보를 심볼 테이블에 저장합니다. 정리하자면 AST는 코드의 대한 변수, 함수, 클래스 정보를 Symbol table은 그 코드에 대한 이름, 타입, 유효 범위(scope), 선언 위치를 가지고 있습니다.

<details>
<summary>Symbol 종류</summary>
<pre>
    const enum SymbolFlags {
        None                    = 0,
        FunctionScopedVariable  = 1 << 0,   // Variable (var) or parameter
        BlockScopedVariable     = 1 << 1,   // A block-scoped variable (let or const)
        Property                = 1 << 2,   // Property or enum member
        EnumMember              = 1 << 3,   // Enum member
        Function                = 1 << 4,   // Function
        Class                   = 1 << 5,   // Class
        Interface               = 1 << 6,   // Interface
        ConstEnum               = 1 << 7,   // Const enum
        RegularEnum             = 1 << 8,   // Enum
        ValueModule             = 1 << 9,   // Instantiated module
        NamespaceModule         = 1 << 10,  // Uninstantiated module
        TypeLiteral             = 1 << 11,  // Type Literal or mapped type
        ObjectLiteral           = 1 << 12,  // Object Literal
        Method                  = 1 << 13,  // Method
        Constructor             = 1 << 14,  // Constructor
        GetAccessor             = 1 << 15,  // Get accessor
        SetAccessor             = 1 << 16,  // Set accessor
        Signature               = 1 << 17,  // Call, construct, or index signature
        TypeParameter           = 1 << 18,  // Type parameter
        TypeAlias               = 1 << 19,  // Type alias
        ExportValue             = 1 << 20,  // Exported value marker (see comment in declareModuleMember in binder)
        Alias                   = 1 << 21,  // An alias for another symbol (see comment in isAliasSymbolDeclaration in checker)
        Prototype               = 1 << 22,  // Prototype property (no source representation)
        ExportStar              = 1 << 23,  // Export * declaration
        Optional                = 1 << 24,  // Optional property
        Transient               = 1 << 25,  // Transient symbol (created during type check)
        Assignment              = 1 << 26,  // Assignment treated as declaration (eg `this.prop = 1`)
        ModuleExports           = 1 << 27,  // Symbol for CommonJS `module` of `module.exports`
        /** @internal */
        All = FunctionScopedVariable | BlockScopedVariable | Property | EnumMember | Function | Class | Interface | ConstEnum | RegularEnum | ValueModule | NamespaceModule | TypeLiteral
            | ObjectLiteral | Method | Constructor | GetAccessor | SetAccessor | Signature | TypeParameter | TypeAlias | ExportValue | Alias | Prototype | ExportStar | Optional | Transient,
    
        Enum = RegularEnum | ConstEnum,
        Variable = FunctionScopedVariable | BlockScopedVariable,
        Value = Variable | Property | EnumMember | ObjectLiteral | Function | Class | Enum | ValueModule | Method | GetAccessor | SetAccessor,
        Type = Class | Interface | Enum | EnumMember | TypeLiteral | TypeParameter | TypeAlias,
        Namespace = ValueModule | NamespaceModule | Enum,
        Module = ValueModule | NamespaceModule,
        Accessor = GetAccessor | SetAccessor,
    
        // Variables can be redeclared, but can not redeclare a block-scoped declaration with the
        // same name, or any other value that is not a variable, e.g. ValueModule or Class
        FunctionScopedVariableExcludes = Value & ~FunctionScopedVariable,
    
        // Block-scoped declarations are not allowed to be re-declared
        // they can not merge with anything in the value space
        BlockScopedVariableExcludes = Value,
    
        ParameterExcludes = Value,
        PropertyExcludes = None,
        EnumMemberExcludes = Value | Type,
        FunctionExcludes = Value & ~(Function | ValueModule | Class),
        ClassExcludes = (Value | Type) & ~(ValueModule | Interface | Function), // class-interface mergability done in checker.ts
        InterfaceExcludes = Type & ~(Interface | Class),
        RegularEnumExcludes = (Value | Type) & ~(RegularEnum | ValueModule), // regular enums merge only with regular enums and modules
        ConstEnumExcludes = (Value | Type) & ~ConstEnum, // const enums merge only with const enums
        ValueModuleExcludes = Value & ~(Function | Class | RegularEnum | ValueModule),
        NamespaceModuleExcludes = 0,
        MethodExcludes = Value & ~Method,
        GetAccessorExcludes = Value & ~SetAccessor,
        SetAccessorExcludes = Value & ~GetAccessor,
        AccessorExcludes = Value & ~Accessor,
        TypeParameterExcludes = Type & ~TypeParameter,
        TypeAliasExcludes = Type,
        AliasExcludes = Alias,
    
        ModuleMember = Variable | Function | Class | Interface | Enum | Module | TypeAlias | Alias,
    
        ExportHasLocal = Function | Class | Enum | ValueModule,
    
        BlockScoped = BlockScopedVariable | Class | Enum,
    
        PropertyOrAccessor = Property | Accessor,
    
        ClassMember = Method | Accessor | Property,
    
        /** @internal */
        ExportSupportsDefaultModifier = Class | Function | Interface,
    
        /** @internal */
        ExportDoesNotSupportDefaultModifier = ~ExportSupportsDefaultModifier,
    
        /** @internal */
        // The set of things we consider semantically classifiable.  Used to speed up the LS during
        // classification.
        Classifiable = Class | Enum | TypeAlias | Interface | TypeParameter | Module | Alias,
    
        /** @internal */
        LateBindingContainer = Class | Interface | TypeLiteral | ObjectLiteral | Function,
    }
</pre>
</details>
  
    

AST와 심볼 테이블은 서로 보완적인 정보를 제공하는 사이입니다. 이 둘을 이용해 컴파일러가 타입 검사, 최적화, 코드 생성 등의 작업을 수행하게 됩니다.

### Checker

Checker 단계에서는 타입스크립트의 주요 기능 중 하나인 정적 타입 검사가 이루어집니다. 이 단계는 Binder가 생성한 심볼 테이블과 AST를 사용하여 작업이 진행됩니다. Parser 단계에서 1차로 문법 오류를 선별했다면 Checker 단계에서는 각 식별자가 선언된 타입에 따라 올바르게 사용되었는지 확인하고, 잘못된 타입의 값이 할당되거나 반환되는 등의 문제가 없는지 검사합니다.

예로 `const foo: number= 'string'` 코드의 경우 `foo`는 문자열을 할당하려 했지만 `foo`의 타입은 number이므로 Checker는 이를 타입 오류로 판단하고 오류 메세지를 반환합니다. 이 예시 외에도 사용하는 모든 타입에 관한 규칙을 검사합니다.

우리가 타입스크립트를 사용하며 편하다고 느끼는 것들 타입 검사, 추론, 자동 완성 등 대부분은 Checker에 의해 일어납니다. 앞서 설명한 3단계는 Checker를 동작시키기 위한 전처리 과정이라고 생각하면 좋을 것 같습니다. 

<details>
<summary>checker 기능들</summary>
<pre>
interface TypeChecker {
    getTypeOfSymbolAtLocation(symbol: Symbol, node: Node): Type;
    getTypeOfSymbol(symbol: Symbol): Type;
    getDeclaredTypeOfSymbol(symbol: Symbol): Type;
    getPropertiesOfType(type: Type): Symbol[];
    getPropertyOfType(type: Type, propertyName: string): Symbol | undefined;
    getPrivateIdentifierPropertyOfType(leftType: Type, name: string, location: Node): Symbol | undefined;
    /** @internal */ getTypeOfPropertyOfType(type: Type, propertyName: string): Type | undefined;
    getIndexInfoOfType(type: Type, kind: IndexKind): IndexInfo | undefined;
    getIndexInfosOfType(type: Type): readonly IndexInfo[];
    getIndexInfosOfIndexSymbol: (indexSymbol: Symbol) => IndexInfo[];
    getSignaturesOfType(type: Type, kind: SignatureKind): readonly Signature[];
    getIndexTypeOfType(type: Type, kind: IndexKind): Type | undefined;
    /** @internal */ getIndexType(type: Type): Type;
    getBaseTypes(type: InterfaceType): BaseType[];
    getBaseTypeOfLiteralType(type: Type): Type;
    getWidenedType(type: Type): Type;
    /** @internal */
    getPromisedTypeOfPromise(promise: Type, errorNode?: Node): Type | undefined;
    /** @internal */
    getAwaitedType(type: Type): Type | undefined;
    /** @internal */
    isEmptyAnonymousObjectType(type: Type): boolean;
    getReturnTypeOfSignature(signature: Signature): Type;
    /**
     * Gets the type of a parameter at a given position in a signature.
     * Returns `any` if the index is not valid.
     *
     * @internal
     */
    getParameterType(signature: Signature, parameterIndex: number): Type;
    /** @internal */ getParameterIdentifierNameAtPosition(signature: Signature, parameterIndex: number): [parameterName: __String, isRestParameter: boolean] | undefined;
    getNullableType(type: Type, flags: TypeFlags): Type;
    getNonNullableType(type: Type): Type;
    /** @internal */ getNonOptionalType(type: Type): Type;
    /** @internal */ isNullableType(type: Type): boolean;
    getTypeArguments(type: TypeReference): readonly Type[];

    // TODO: GH#18217 `xToDeclaration` calls are frequently asserted as defined.
    /** Note that the resulting nodes cannot be checked. */
    typeToTypeNode(type: Type, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): TypeNode | undefined;
    /** @internal */ typeToTypeNode(type: Type, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined, tracker?: SymbolTracker): TypeNode | undefined; // eslint-disable-line @typescript-eslint/unified-signatures
    /** Note that the resulting nodes cannot be checked. */
    signatureToSignatureDeclaration(signature: Signature, kind: SyntaxKind, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): SignatureDeclaration & {typeArguments?: NodeArray<TypeNode>} | undefined;
    /** @internal */ signatureToSignatureDeclaration(signature: Signature, kind: SyntaxKind, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined, tracker?: SymbolTracker): SignatureDeclaration & {typeArguments?: NodeArray<TypeNode>} | undefined; // eslint-disable-line @typescript-eslint/unified-signatures
    /** Note that the resulting nodes cannot be checked. */
    indexInfoToIndexSignatureDeclaration(indexInfo: IndexInfo, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): IndexSignatureDeclaration | undefined;
    /** @internal */ indexInfoToIndexSignatureDeclaration(indexInfo: IndexInfo, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined, tracker?: SymbolTracker): IndexSignatureDeclaration | undefined; // eslint-disable-line @typescript-eslint/unified-signatures
    /** Note that the resulting nodes cannot be checked. */
    symbolToEntityName(symbol: Symbol, meaning: SymbolFlags, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): EntityName | undefined;
    /** Note that the resulting nodes cannot be checked. */
    symbolToExpression(symbol: Symbol, meaning: SymbolFlags, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): Expression | undefined;
    /**
     * Note that the resulting nodes cannot be checked.
     *
     * @internal
     */
    symbolToNode(symbol: Symbol, meaning: SymbolFlags, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): Node | undefined;
    /** Note that the resulting nodes cannot be checked. */
    symbolToTypeParameterDeclarations(symbol: Symbol, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): NodeArray<TypeParameterDeclaration> | undefined;
    /** Note that the resulting nodes cannot be checked. */
    symbolToParameterDeclaration(symbol: Symbol, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): ParameterDeclaration | undefined;
    /** Note that the resulting nodes cannot be checked. */
    typeParameterToDeclaration(parameter: TypeParameter, enclosingDeclaration: Node | undefined, flags: NodeBuilderFlags | undefined): TypeParameterDeclaration | undefined;

    getSymbolsInScope(location: Node, meaning: SymbolFlags): Symbol[];
    getSymbolAtLocation(node: Node): Symbol | undefined;
    /** @internal */ getIndexInfosAtLocation(node: Node): readonly IndexInfo[] | undefined;
    getSymbolsOfParameterPropertyDeclaration(parameter: ParameterDeclaration, parameterName: string): Symbol[];
    /**
     * The function returns the value (local variable) symbol of an identifier in the short-hand property assignment.
     * This is necessary as an identifier in short-hand property assignment can contains two meaning: property name and property value.
     */
    getShorthandAssignmentValueSymbol(location: Node | undefined): Symbol | undefined;

    getExportSpecifierLocalTargetSymbol(location: ExportSpecifier | Identifier): Symbol | undefined;
    /**
     * If a symbol is a local symbol with an associated exported symbol, returns the exported symbol.
     * Otherwise returns its input.
     * For example, at `export type T = number;`:
     *     - `getSymbolAtLocation` at the location `T` will return the exported symbol for `T`.
     *     - But the result of `getSymbolsInScope` will contain the *local* symbol for `T`, not the exported symbol.
     *     - Calling `getExportSymbolOfSymbol` on that local symbol will return the exported symbol.
     */
    getExportSymbolOfSymbol(symbol: Symbol): Symbol;
    getPropertySymbolOfDestructuringAssignment(location: Identifier): Symbol | undefined;
    getTypeOfAssignmentPattern(pattern: AssignmentPattern): Type;
    getTypeAtLocation(node: Node): Type;
    getTypeFromTypeNode(node: TypeNode): Type;

    signatureToString(signature: Signature, enclosingDeclaration?: Node, flags?: TypeFormatFlags, kind?: SignatureKind): string;
    typeToString(type: Type, enclosingDeclaration?: Node, flags?: TypeFormatFlags): string;
    symbolToString(symbol: Symbol, enclosingDeclaration?: Node, meaning?: SymbolFlags, flags?: SymbolFormatFlags): string;
    typePredicateToString(predicate: TypePredicate, enclosingDeclaration?: Node, flags?: TypeFormatFlags): string;

    /** @internal */ writeSignature(signature: Signature, enclosingDeclaration?: Node, flags?: TypeFormatFlags, kind?: SignatureKind, writer?: EmitTextWriter): string;
    /** @internal */ writeType(type: Type, enclosingDeclaration?: Node, flags?: TypeFormatFlags, writer?: EmitTextWriter): string;
    /** @internal */ writeSymbol(symbol: Symbol, enclosingDeclaration?: Node, meaning?: SymbolFlags, flags?: SymbolFormatFlags, writer?: EmitTextWriter): string;
    /** @internal */ writeTypePredicate(predicate: TypePredicate, enclosingDeclaration?: Node, flags?: TypeFormatFlags, writer?: EmitTextWriter): string;

    getFullyQualifiedName(symbol: Symbol): string;
    getAugmentedPropertiesOfType(type: Type): Symbol[];

    getRootSymbols(symbol: Symbol): readonly Symbol[];
    getSymbolOfExpando(node: Node, allowDeclaration: boolean): Symbol | undefined;
    getContextualType(node: Expression): Type | undefined;
    /** @internal */ getContextualType(node: Expression, contextFlags?: ContextFlags): Type | undefined; // eslint-disable-line @typescript-eslint/unified-signatures
    /** @internal */ getContextualTypeForObjectLiteralElement(element: ObjectLiteralElementLike): Type | undefined;
    /** @internal */ getContextualTypeForArgumentAtIndex(call: CallLikeExpression, argIndex: number): Type | undefined;
    /** @internal */ getContextualTypeForJsxAttribute(attribute: JsxAttribute | JsxSpreadAttribute): Type | undefined;
    /** @internal */ isContextSensitive(node: Expression | MethodDeclaration | ObjectLiteralElementLike | JsxAttributeLike): boolean;
    /** @internal */ getTypeOfPropertyOfContextualType(type: Type, name: __String): Type | undefined;

    /**
     * returns unknownSignature in the case of an error.
     * returns undefined if the node is not valid.
     * @param argumentCount Apparent number of arguments, passed in case of a possibly incomplete call. This should come from an ArgumentListInfo. See `signatureHelp.ts`.
     */
    getResolvedSignature(node: CallLikeExpression, candidatesOutArray?: Signature[], argumentCount?: number): Signature | undefined;
    /** @internal */ getResolvedSignatureForSignatureHelp(node: CallLikeExpression, candidatesOutArray?: Signature[], argumentCount?: number): Signature | undefined;
    /** @internal */ getResolvedSignatureForStringLiteralCompletions(call: CallLikeExpression, editingArgument: Node, candidatesOutArray: Signature[]): Signature | undefined;
    /** @internal */ getExpandedParameters(sig: Signature): readonly (readonly Symbol[])[];
    /** @internal */ hasEffectiveRestParameter(sig: Signature): boolean;
    /** @internal */ containsArgumentsReference(declaration: SignatureDeclaration): boolean;

    getSignatureFromDeclaration(declaration: SignatureDeclaration): Signature | undefined;
    isImplementationOfOverload(node: SignatureDeclaration): boolean | undefined;
    isUndefinedSymbol(symbol: Symbol): boolean;
    isArgumentsSymbol(symbol: Symbol): boolean;
    isUnknownSymbol(symbol: Symbol): boolean;
    /** @internal */ getMergedSymbol(symbol: Symbol): Symbol;

    getConstantValue(node: EnumMember | PropertyAccessExpression | ElementAccessExpression): string | number | undefined;
    isValidPropertyAccess(node: PropertyAccessExpression | QualifiedName | ImportTypeNode, propertyName: string): boolean;
    /**
     * Exclude accesses to private properties.
     *
     * @internal
     */
    isValidPropertyAccessForCompletions(node: PropertyAccessExpression | ImportTypeNode | QualifiedName, type: Type, property: Symbol): boolean;
    /** Follow all aliases to get the original symbol. */
    getAliasedSymbol(symbol: Symbol): Symbol;
    /** Follow a *single* alias to get the immediately aliased symbol. */
    getImmediateAliasedSymbol(symbol: Symbol): Symbol | undefined;
    getExportsOfModule(moduleSymbol: Symbol): Symbol[];
    /**
     * Unlike `getExportsOfModule`, this includes properties of an `export =` value.
     *
     * @internal
     */
    getExportsAndPropertiesOfModule(moduleSymbol: Symbol): Symbol[];
    /** @internal */ forEachExportAndPropertyOfModule(moduleSymbol: Symbol, cb: (symbol: Symbol, key: __String) => void): void;
    getJsxIntrinsicTagNamesAt(location: Node): Symbol[];
    isOptionalParameter(node: ParameterDeclaration): boolean;
    getAmbientModules(): Symbol[];

    tryGetMemberInModuleExports(memberName: string, moduleSymbol: Symbol): Symbol | undefined;
    /**
     * Unlike `tryGetMemberInModuleExports`, this includes properties of an `export =` value.
     * Does *not* return properties of primitive types.
     *
     * @internal
     */
    tryGetMemberInModuleExportsAndProperties(memberName: string, moduleSymbol: Symbol): Symbol | undefined;
    getApparentType(type: Type): Type;
    /** @internal */ getSuggestedSymbolForNonexistentProperty(name: MemberName | string, containingType: Type): Symbol | undefined;
    /** @internal */ getSuggestedSymbolForNonexistentJSXAttribute(name: Identifier | string, containingType: Type): Symbol | undefined;
    /** @internal */ getSuggestionForNonexistentProperty(name: MemberName | string, containingType: Type): string | undefined;
    /** @internal */ getSuggestedSymbolForNonexistentSymbol(location: Node, name: string, meaning: SymbolFlags): Symbol | undefined;
    /** @internal */ getSuggestionForNonexistentSymbol(location: Node, name: string, meaning: SymbolFlags): string | undefined;
    /** @internal */ getSuggestedSymbolForNonexistentModule(node: Identifier, target: Symbol): Symbol | undefined;
    /** @internal */ getSuggestedSymbolForNonexistentClassMember(name: string, baseType: Type): Symbol | undefined;
    /** @internal */ getSuggestionForNonexistentExport(node: Identifier, target: Symbol): string | undefined;
    getBaseConstraintOfType(type: Type): Type | undefined;
    getDefaultFromTypeParameter(type: Type): Type | undefined;

    /**
     * Gets the intrinsic `any` type. There are multiple types that act as `any` used internally in the compiler,
     * so the type returned by this function should not be used in equality checks to determine if another type
     * is `any`. Instead, use `type.flags & TypeFlags.Any`.
     */
    getAnyType(): Type;
    getStringType(): Type;
    getStringLiteralType(value: string): StringLiteralType;
    getNumberType(): Type;
    getNumberLiteralType(value: number): NumberLiteralType;
    getBigIntType(): Type;
    getBooleanType(): Type;
    /* eslint-disable @typescript-eslint/unified-signatures */
    /** @internal */
    getFalseType(fresh?: boolean): Type;
    getFalseType(): Type;
    /** @internal */
    getTrueType(fresh?: boolean): Type;
    getTrueType(): Type;
    /* eslint-enable @typescript-eslint/unified-signatures */
    getVoidType(): Type;
    /**
     * Gets the intrinsic `undefined` type. There are multiple types that act as `undefined` used internally in the compiler
     * depending on compiler options, so the type returned by this function should not be used in equality checks to determine
     * if another type is `undefined`. Instead, use `type.flags & TypeFlags.Undefined`.
     */
    getUndefinedType(): Type;
    /**
     * Gets the intrinsic `null` type. There are multiple types that act as `null` used internally in the compiler,
     * so the type returned by this function should not be used in equality checks to determine if another type
     * is `null`. Instead, use `type.flags & TypeFlags.Null`.
     */
    getNullType(): Type;
    getESSymbolType(): Type;
    /**
     * Gets the intrinsic `never` type. There are multiple types that act as `never` used internally in the compiler,
     * so the type returned by this function should not be used in equality checks to determine if another type
     * is `never`. Instead, use `type.flags & TypeFlags.Never`.
     */
    getNeverType(): Type;
    /** @internal */ getOptionalType(): Type;
    /** @internal */ getUnionType(types: Type[], subtypeReduction?: UnionReduction): Type;
    /** @internal */ createArrayType(elementType: Type): Type;
    /** @internal */ getElementTypeOfArrayType(arrayType: Type): Type | undefined;
    /** @internal */ createPromiseType(type: Type): Type;
    /** @internal */ getPromiseType(): Type;
    /** @internal */ getPromiseLikeType(): Type;
    /** @internal */ getAsyncIterableType(): Type | undefined;

    /** @internal */ isTypeAssignableTo(source: Type, target: Type): boolean;
    /** @internal */ createAnonymousType(symbol: Symbol | undefined, members: SymbolTable, callSignatures: Signature[], constructSignatures: Signature[], indexInfos: IndexInfo[]): Type;
    /** @internal */ createSignature(
        declaration: SignatureDeclaration | undefined,
        typeParameters: readonly TypeParameter[] | undefined,
        thisParameter: Symbol | undefined,
        parameters: readonly Symbol[],
        resolvedReturnType: Type,
        typePredicate: TypePredicate | undefined,
        minArgumentCount: number,
        flags: SignatureFlags
    ): Signature;
    /** @internal */ createSymbol(flags: SymbolFlags, name: __String): TransientSymbol;
    /** @internal */ createIndexInfo(keyType: Type, type: Type, isReadonly: boolean, declaration?: SignatureDeclaration): IndexInfo;
    /** @internal */ isSymbolAccessible(symbol: Symbol, enclosingDeclaration: Node | undefined, meaning: SymbolFlags, shouldComputeAliasToMarkVisible: boolean): SymbolAccessibilityResult;
    /** @internal */ tryFindAmbientModule(moduleName: string): Symbol | undefined;
    /** @internal */ tryFindAmbientModuleWithoutAugmentations(moduleName: string): Symbol | undefined;

    /** @internal */ getSymbolWalker(accept?: (symbol: Symbol) => boolean): SymbolWalker;

    // Should not be called directly.  Should only be accessed through the Program instance.
    /** @internal */ getDiagnostics(sourceFile?: SourceFile, cancellationToken?: CancellationToken): Diagnostic[];
    /** @internal */ getGlobalDiagnostics(): Diagnostic[];
    /** @internal */ getEmitResolver(sourceFile?: SourceFile, cancellationToken?: CancellationToken): EmitResolver;

    /** @internal */ getNodeCount(): number;
    /** @internal */ getIdentifierCount(): number;
    /** @internal */ getSymbolCount(): number;
    /** @internal */ getTypeCount(): number;
    /** @internal */ getInstantiationCount(): number;
    /** @internal */ getRelationCacheSizes(): { assignable: number, identity: number, subtype: number, strictSubtype: number };
    /** @internal */ getRecursionIdentity(type: Type): object | undefined;
    /** @internal */ getUnmatchedProperties(source: Type, target: Type, requireOptionalProperties: boolean, matchDiscriminantProperties: boolean): IterableIterator<Symbol>;

    /**
     * True if this type is the `Array` or `ReadonlyArray` type from lib.d.ts.
     * This function will _not_ return true if passed a type which
     * extends `Array` (for example, the TypeScript AST's `NodeArray` type).
     */
    isArrayType(type: Type): boolean;
    /**
     * True if this type is a tuple type. This function will _not_ return true if
     * passed a type which extends from a tuple.
     */
    isTupleType(type: Type): boolean;
    /**
     * True if this type is assignable to `ReadonlyArray<any>`.
     */
    isArrayLikeType(type: Type): boolean;

    /**
     * True if `contextualType` should not be considered for completions because
     * e.g. it specifies `kind: "a"` and obj has `kind: "b"`.
     *
     * @internal
     */
    isTypeInvalidDueToUnionDiscriminant(contextualType: Type, obj: ObjectLiteralExpression | JsxAttributes): boolean;
    /** @internal */ getExactOptionalProperties(type: Type): Symbol[];
    /**
     * For a union, will include a property if it's defined in *any* of the member types.
     * So for `{ a } | { b }`, this will include both `a` and `b`.
     * Does not include properties of primitive types.
     *
     * @internal
     */
    getAllPossiblePropertiesOfTypes(type: readonly Type[]): Symbol[];
    /** @internal */ resolveName(name: string, location: Node | undefined, meaning: SymbolFlags, excludeGlobals: boolean): Symbol | undefined;
    /** @internal */ getJsxNamespace(location?: Node): string;
    /** @internal */ getJsxFragmentFactory(location: Node): string | undefined;

    /**
     * Note that this will return undefined in the following case:
     *     // a.ts
     *     export namespace N { export class C { } }
     *     // b.ts
     *     <<enclosingDeclaration>>
     * Where `C` is the symbol we're looking for.
     * This should be called in a loop climbing parents of the symbol, so we'll get `N`.
     *
     * @internal
     */
    getAccessibleSymbolChain(symbol: Symbol, enclosingDeclaration: Node | undefined, meaning: SymbolFlags, useOnlyExternalAliasing: boolean): Symbol[] | undefined;
    getTypePredicateOfSignature(signature: Signature): TypePredicate | undefined;
    /** @internal */ resolveExternalModuleName(moduleSpecifier: Expression): Symbol | undefined;
    /**
     * An external module with an 'export =' declaration resolves to the target of the 'export =' declaration,
     * and an external module with no 'export =' declaration resolves to the module itself.
     *
     * @internal
     */
    resolveExternalModuleSymbol(symbol: Symbol): Symbol;
    /**
     * @param node A location where we might consider accessing `this`. Not necessarily a ThisExpression.
     *
     * @internal
     */
    tryGetThisTypeAt(node: Node, includeGlobalThis?: boolean, container?: ThisContainer): Type | undefined;
    /** @internal */ getTypeArgumentConstraint(node: TypeNode): Type | undefined;

    /**
     * Does *not* get *all* suggestion diagnostics, just the ones that were convenient to report in the checker.
     * Others are added in computeSuggestionDiagnostics.
     *
     * @internal
     */
    getSuggestionDiagnostics(file: SourceFile, cancellationToken?: CancellationToken): readonly DiagnosticWithLocation[];

    /**
     * Depending on the operation performed, it may be appropriate to throw away the checker
     * if the cancellation token is triggered. Typically, if it is used for error checking
     * and the operation is cancelled, then it should be discarded, otherwise it is safe to keep.
     */
    runWithCancellationToken<T>(token: CancellationToken, cb: (checker: TypeChecker) => T): T;

    /** @internal */ getLocalTypeParametersOfClassOrInterfaceOrTypeAlias(symbol: Symbol): readonly TypeParameter[] | undefined;
    /** @internal */ isDeclarationVisible(node: Declaration | AnyImportSyntax): boolean;
    /** @internal */ isPropertyAccessible(node: Node, isSuper: boolean, isWrite: boolean, containingType: Type, property: Symbol): boolean;
    /** @internal */ getTypeOnlyAliasDeclaration(symbol: Symbol): TypeOnlyAliasDeclaration | undefined;
    /** @internal */ getMemberOverrideModifierStatus(node: ClassLikeDeclaration, member: ClassElement, memberSymbol: Symbol): MemberOverrideStatus;
    /** @internal */ isTypeParameterPossiblyReferenced(tp: TypeParameter, node: Node): boolean;
    /** @internal */ typeHasCallOrConstructSignatures(type: Type): boolean;
}
</pre>
</details>
  
    

### Emiiter

Emiiter단계에서는 Checker에 의해 검증된 타입스크립트 코드를 기반으로 자바스크립트 코드로 변환되는 과정을 담은 단계입니다. 이 과정에서 Emiiter는 자바스크립트에 필요 없는 타입스크립트 구문과 기능들(type, interface, enum 등)을 나누고 필요에 따라 완전히 제거하고 이로서 순수한 자바스크립트 코드로 변환됩니다.

## item 1: 타입스크립트와 자바스크립트의 차이


누가 봐도 타입스크립트는 자바스크립트의 상위 집합으로 볼 수 있습니다. 타입스크립트는 자바스크립트의 모든 기능을 포함하고 그 이상의 타입과 추가적인 기능을 제공하기 때문입니다. 하지만 이것은 표면적의 차이일 뿐 더 중요한 차이는 내부적인 컴파일 프로세스가 다르다는 점이라고 생각합니다. 자바스크립트도 코드를 Tokenizing하고 그것을 AST로 parse하고 Interpreter에 의해 Bytecode로 변환되고 이후 브라우저 엔진의 컴파일러에 의해 브라우저에서 돌아갑니다. 타입스크립트는 이 과정에서 타입을 생성하고 검사하는 프로세스를 추가 했습니다. 이 프로세스가 둘의 가장 큰 차이라고 생각합니다. 이런 차이로 인해 우리는 타입을 사용할 수 있고, 코드가 런타임 환경에서 실행되기 전에 문법 오류, 타입 오류를 발견할 수 있으며 타입 추론과 자동 완성을 가능하게 해주기 때문입니다. 그래서 만약 누군가 타입스크립트와 자바스크립트의 차이를 묻는다면 표면적인 차이만 말할 것이 아니라 내부적인 차이로 인해 표면적인 차이가 생긴다고 설명할 것 같습니다.

## item 2: 타입스크립트 설정 이해하기


`tsconfig.json`을 이용해 옵션을 커스텀하게 설정 가능합니다. 이 옵션들은 컴파일러 프로세스 전범위에 영향을 끼치게 됩니다. [tsconfig options](https://typescript-kr.github.io/pages/compiler-options.html)

예를 들어 `target` 옵션은 Scanner, Parser 단계에서 어떤 문법으로 scan하고 parse 해야 하는지 정합니다. ES5로 설정되어 있다면 ES5 문법을 인식하고 ES6으로 설정되어 있다면 ES6 문법을 인식해 프로세스를 진행합니다. 타입과 관련된 옵션들은 대부분 Checker 단계에 활용됩니다. 책에 나온 `noImplicitAny`**,** `strickNullChecks`도 여기서 활용됩니다. 컴파일 결과에 대한 옵션들(module, target, jsx 등)은 Emiiter 단계에서 활용되겠죠 

```tsx
// Checker.ts
var compilerOptions = host.getCompilerOptions();
var languageVersion = getEmitScriptTarget(compilerOptions);
var moduleKind = getEmitModuleKind(compilerOptions);
var legacyDecorators = !!compilerOptions.experimentalDecorators;
var useDefineForClassFields = getUseDefineForClassFields(compilerOptions);
var allowSyntheticDefaultImports = getAllowSyntheticDefaultImports(compilerOptions);
var strictNullChecks = getStrictOptionValue(compilerOptions, "strictNullChecks");
var strictFunctionTypes = getStrictOptionValue(compilerOptions, "strictFunctionTypes");
var strictBindCallApply = getStrictOptionValue(compilerOptions, "strictBindCallApply");
var strictPropertyInitialization = getStrictOptionValue(compilerOptions, "strictPropertyInitialization");
var noImplicitAny = getStrictOptionValue(compilerOptions, "noImplicitAny");
var noImplicitThis = getStrictOptionValue(compilerOptions, "noImplicitThis");
var useUnknownInCatchVariables = getStrictOptionValue(compilerOptions, "useUnknownInCatchVariables");
var keyofStringsOnly = !!compilerOptions.keyofStringsOnly;
var defaultIndexFlags = keyofStringsOnly ? IndexFlags.StringsOnly : IndexFlags.None;
var freshObjectLiteralFlag = compilerOptions.suppressExcessPropertyErrors ? 0 : ObjectFlags.FreshLiteral;
var exactOptionalPropertyTypes = compilerOptions.exactOptionalPropertyTypes;
```

## item 3: 코드 생성과 타입은 관계가 없습니다.


책에는 타입스크립트 컴파일러가 두 가지 역할을 수행한다고 적혀있습니다.

1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일 합니다. = Emiiter
2. 코드의 타입 오류를 체크합니다. = Checker

이 둘은 독립적입니다. Checker에서 타입 에러가 생겼더라도 Emiiter에서는 이를 무시하고 JS로 변환 시킵니다. 이유는 Emiiter 단계에서 타입과 관련된 코드를 걷어내기 때문입니다. 그렇기 때문에 자바스크립트 실행 시점에는 에러가 없는 상태가 되겠죠 하지만 실행되고 나서는 코드가 돌아가지 않을 수 있습니다.

## item 4: 구조적 타이핑


구조적 타이핑은 Checker 단계에서 이루어집니다. Checker 단계에서는 타입의 호환성을 체크하는데 이 때 구조적 타이핑을 기반으로해 타입, 인터페이스나 클래스의 구조를 비교하고 호환성을 판단합니다.

```tsx
interface Vector2D {
	x: number;
	y: number;
}

interface NamedVector {
	name: string;
	x: number;
	y: number;
}

funciotn calculateLength(v: Vector2D) {
	return Math.sqrt(v.x * v.x + v.y * v.y);
}

const v: NamedVector = { ... }
calculateLength(v) // 정상
```

calculateLength의 인자 v는 Vector2D 타입을 인자로 받지만 NamedVector 타입을 넘겨줘도 아무런 이상이 없습니다. Checker는 이를 감지하고 서로의 호환성을 체크해 NamedVector가 Vector2D에 호환이 되기 때문에 문제를 삼지 않고 넘어갔기 때문입니다. 

구조적 타이핑은 타입스크립트의 핵심 특징 중 하나입니다. 이를 의도적으로 활용하면 다른 타입간에 코드를 재사용할 수 있고 타입을 더 유연하게 활용할 수 있게 됩니다. 하지만 가독성을 떨어트릴 수 있습니다. 전혀 상관없는 두 타입이 호환이 된다는 이유로 구조적 타이핑을 이용하면 코드를 읽는 사람은 이를 이해하기 위해서 더 많은 노력이 필요할 수 있습니다. 

저는 구조적 타이핑이 좋은 것인지 의문이 듭니다. 유연하고 편하다는 장점이 있지만 이는 any 타입도 마찬가지입니다. 단점도 any와 비교하면 가독성이 떨어지고 코드에 혼란을 줄 수 있다는 단점도 일치합니다. 구조적 타이핑을 의도적으로 활용하는 것이 올바른 것인지 모르겠습니다.

## item 5: any 타입 지양하기


 any는 Checker에서 타입 체크를 수행하지 않습니다. 즉 any는 프리 패스 타입이라고 보면 될 것 같습니다. 이런 any는 유연하고 편리하다는 장점이 있지만 이건 달콤한 독이라 생각합니다. any는 Checker 단계를 회피해 아래와 같은 단점들이 생깁니다.

****any 타입은 타입 안정성이 없습니다.****

위의 설명대로 any는 타입 체크를 회피합니다. 그렇기 때문에 any 타입의 변수에 어떤 값을 넣던 Checker는 알 수 없습니다.

```tsx
const name: string = 10 as any
```

만약 as 문을 이용해 any 타입을 지정해 줬다면 name는 any가 되고 어느 타입이 들어가도 Checker가 걸러주지 않게 되어 오류를 표시하지 않습니다. 만약 어느 곳에서 `name.toLowerCase()`를 사용하고 있는데 any로 인해 number를 넣게 된다면 런타임 환경에서 오류를 확인해야 합니다.

****************************************************************************************************any는 함수 시그니처를 무시해 버립니다.****************************************************************************************************

```tsx
const foo = (v: number): number => v
const birthDate: any = '1000-01-01'
foo(birthDate) // 정상
```

마찬가지로 Checker가 any 타입을 무시해 foo에 대한 타입 체크가 정상적으로 이루어지지 않았습니다. 

****any 타입에는 언어 서비스가 적용되지 않습니다.****

언어 서비스는 Checker에 의해 결정됩니다. 하지만 any는 Checker가 회피하게 되어 언어 서비스에 필요한 정보들을 만들어줄 수 없습니다.

이 외에도 Checker에게 검사를 받지 못해 발견될 수 있는 버그를 발견하지 못하는 일 등 여러 일들이 발생할 수 있습니다 그렇기 때문에 any는 정말로 필요한 상황이 아니라면 꼭 지양해야 하는 것이 올바른 것 같습니다.

## item 8: 타입 공간과 값 공간의 심벌 구분하기

```tsx
type Text = 'string'
const Text = 'text'
```

타입스크립트는 타입과 값의 이름을 동일하게 사용할 수 있습니다. 이게 어떻게 가능한지는 Scanner, Parser, Binder를 이해하면 알 수 있습니다. Scanner에서 토큰화를 진행할 때 선언된 타입은 고려되지 않습니다. 역시 AST를 만드는 Parser에서도 선언된 타입 정보는 접근할 수 없습니다. 선언된 타입 정보는 Binder에 의해 타입이 선언됩니다. AST와 Symbol table은 서로 독립적인 존재이기 때문에 같은 이름을 써도 아무런 문제가 생기지 않습니다. (추정)

다만 같은 이름을 쓰면 다음과 같은 문제가 발생할 수 있습니다.

```tsx
interface Cylinder {
	radius: number;
}

const Cylinder = (radius: number) => ({radius})

const calculateVolume = (shape: unkown) => {
	if(shape instanceof Cylinder) {
		shape.radius // {} 형식에 radius가 없습니다.
	}
}
```

instanceof을 이용해 Cylinder인지 체크하려 하지만 instanceof는 자바스크립트의 런타임 연산자입니다. 타입스크립트는 Emiiter에 의해 런타임 환경 전에 모든 타입이 사라집니다. 때문에 instanceof로 비교한 것은 타입 Cylinder이 아닌 함수 Cylinder가 됩니다. 이렇게 같은 이름을 작성했을 경우 예상치 못한 에러가 생길 수 있습니다. AST와 Symbol의 생명 주기가 다름을 이해한다면 같은 이름을 허용한다 하더라도 지양해야 한다는 것을 알 수 있습니다.
