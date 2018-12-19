---
layout: page
title:  "Unit tests for ANTLR Lexer"
date:   2018-12-18 00:00:00 -0300
categories: antlr java compiler test junit
---

> From ANTLR website: "ANTLR (ANother Tool for Language Recognition) is a powerful parser generator for reading, processing, executing, or translating structured text or binary files. It's widely used to build languages, tools, and frameworks. From a grammar, ANTLR generates a parser that can build and walk parse trees."

## Unit tests for ANTLR Lexer

ANTLR is such a great tool for building a DSL - Domain-Specific Language (or our own language). From a text file, called the grammar, it generates the classes responsible for parsing and walking through the language.  

The grammar is composed of two types of rules:  

* Lexer
* Parser

There are several interesting articles about using ANTLR. Finding something useful about how to unit test it seems pretty hard though.  
And as of any component of a Software, it is important to test each part of a given language. Especially complex components as a custom language.  

Most articles out there shows only how to test the listener or the integrated solution.  

This article shows a way make a unit test for the Lexer.  

## Lexer

The *Lexer* part is responsible for converting a character or string into tokens. These rules are the first ones to be processed.  
Lexer rules are described as LOWER_CASE. They might be either in the same files as parser rules or a separate file.  

For instance, for the rule `BOOLEAN : 'true' | 'false' ;`, when the Lexer processes a text file, when it matches a *true* or a *false* string, it will turn it into a *BOOLEAN* token.  

More Examples:  

        OPEN_PAR : '(' ;
        CLOSE_PAR : ')' ;

        OPEN_BRACE : '{' ;
        CLOSE_BRACE : '}' ;

        BOOLEAN : 'true' | 'false' ;

        fragment LOWERCASE  : [a-z] ;
        fragment UPPERCASE  : [A-Z] ;

        WORD                : (LOWERCASE | UPPERCASE | '_')+ ;


Therefore, testing a Lexer is meant to **check whether a text is interpreted as the expected token sequence**.  


## Reading and Listening to errors

> Code samples here are made using Kotlin. As it runs over the Jvm, they are similar to Java. So it should be easily understandable by Java developers.  

At first, let's create a method with some boilerplate code to invoke the Lexer processing:  

        private fun getTokensFromText(txt: String): List<Token> {
            val iStream = ByteArrayInputStream(txt.toByteArray())
            val cStream = CharStreams.fromStream(iStream)
            val lex = MyLexer(cStream)
            lex.addErrorListener(errorListener)     // listener
            val tokenStream = CommonTokenStream(lex)
            tokenStream.fill()
            return tokenStream.tokens
        }


This accepts a string (*txt*) and processes it into MyLexer.  
Then we add an error listener to be aware of what may have happened in case of error.  Without this, *ANTLR* would not say what is wrong. 
This listener is just an implementation of `ANTLRErrorListener`.  
The method returns a list of tokens.  


## The tests

Given the above method, now we just need to test some *strings* and check the tokens got as the output.  

    fun testFunction() {
        val tokens = getTokensFromText("function ()")

        assertEquals(4, tokens.size) // includes EOF
        assertEquals(MplLexer.WORD, tokens[0].type)
        assertEquals(MplLexer.OPEN_PAR, tokens[1].type)
        assertEquals(MplLexer.CLOSE_PAR, tokens[2].type)
    }

## Final notes

The most common mistake in Lexer when write some grammar is leaving some **ambiguity**. With this code, it's possible and easy to verify this problem.  

> REMEBER: ANTLR Lexer tries to match the largest possible token

This article is based on my open source project **Tevim**. Check out:  

* The [Lexer grammar](https://github.com/ssricardo/tevim/blob/master/source/src/main/java/org/rss/tools/tevim/parsing/grammar2/MplLexer.g4)
* The [LexerTest](https://github.com/ssricardo/tevim/blob/master/source/src/test/kotlin/org/rss/tools/tevim/test/language/LexerTest.kt)

*!EOF*

---