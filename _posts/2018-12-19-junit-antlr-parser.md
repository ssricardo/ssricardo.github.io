---
layout: page
title:  "Unit tests for ANTLR parser"
date:   2018-12-18 00:00:00 -0300
categories: antlr java compiler parser test
---

> Take a look into the previous post (Lexer test) if you haven't yet

As said in the previous post, it's hard to find resources about how to make unit tests for ANTLR components.  
In that article, it was shown a way to test the *Lexer* of an ANTLR grammar. Here, we show a way to test the ***Parser* rules.  

## Parser

In ANTLR (and likely any language processor), the parser is responsible for getting a sequence of tokens and trying to match them against its rules. These tokens come from the *Lexer* (which should have run before the parser).  

The Parser rules are the ones starting with a lowercase letter, for instance: `document: (function)+ ;`.  
Another example, based on existing Lexer rules (tokens): `singleParam : (WORD EQUALS)? paramValue ;`.  

Considering we are making a unit test for the Parser, it makes no sense testing the Lexer again as well. If we test them integrated, when we get an error it would be harder to know whether the error is on the Parser or on the Lexer.  
Integrated tests are useful in other situations, but that's not the case here. 

## Creating the Parser

> Again...code samples here are made using Kotlin. As it runs over the Jvm, they are similar to Java. So it should be easily understandable by Java developers.  

We'll use a method for creating Parser instances based on a list of Tokens:  

        private fun createParserNoError(tokens: List<TestToken>): MyParser {
            val ts = ListTokenSource(tokens)
            val c = CommonTokenStream(ts)
            val p = MyParser(c)
            p.addErrorListener(NoErrorListener)
            return p
        }

We are adding an *ErrorListener* which is an implementation of `BaseErrorListener`.  
As we do not expect any error in this test, within this *listener* we call Junit's `Assert.fail()`.  

Here the listener is:  

        object NoErrorListener: BaseErrorListener() {

            override fun syntaxError(recognizer: Recognizer<*, *>?, offendingSymbol: Any?, line: Int, charPositionInLine: Int, msg: String?, e: RecognitionException?) {
                fail()
            }

            override fun reportAmbiguity(recognizer: Parser?, dfa: DFA?, startIndex: Int, stopIndex: Int, exact: Boolean, ambigAlts: BitSet?, configs: ATNConfigSet?) {
                fail()
            }
        }

If you are using Java or any other language, don't about Kotlin's specific syntax. Just use the same ANTLR APIs.  

## Parser test

You may have noticed the *TestToken* on `createParserNoError` parameter (above).  
As we don't want to test the Lexer along with the Parser, we are going to provide the token list for the test (without calling the Lexer).  

*Token* is an interface of ANTLR. Therefore we need to use an implementation of it.  
For testing purposes, a *class TestToken* was created. It isn't necessary to use most of *Token* methods. Thus, they may just return 0 or null.  Two methods are needed: `getText()` and `getType()`. To provide data for them, we add parameters to the constructor.  

This is the **TestToken**:  

        // class and constructor declarations
        class TestToken (val _text: String, val tcod: Int) : Token {

            override fun getText(): String? {
                return _text
            }

            override fun getType(): Int {
                return tcod
            }

            override fun getLine(): Int = 0


            override fun getCharPositionInLine(): Int = 0

            override fun getChannel(): Int = 0

            override fun getTokenIndex(): Int = 0

            override fun getStartIndex(): Int = 0

            override fun getStopIndex(): Int = 0

            override fun getTokenSource(): TokenSource? {
                return null
            }

            override fun getInputStream(): CharStream? {
                return null
            }
        }

### Test cases

Now, in each test we need to **provide a list of tokens** and **check** whether the parser **matches the correct rule**.  

Example:  

        @Test 
        fun testFunctionPlain() {
            val noParam = createParserNoError(listOf(
                    /*new */ TestToken("count", MplLexer.WORD),
                    /*new */ TestToken("", MplLexer.NEWLINE)))

            val f = noParam.function()
            assertTrue("count" in f.text)
        }

## See Also


This article is based on my open source project **Tevim**. Check out:  

* [Parser Grammar](https://github.com/ssricardo/tevim/blob/master/source/src/main/java/org/rss/tools/tevim/parsing/grammar2/Mpl.g4)
* [Parser Test](https://github.com/ssricardo/tevim/blob/master/source/src/test/kotlin/org/rss/tools/tevim/test/language/ParserTest.kt)


*!EOF*

---