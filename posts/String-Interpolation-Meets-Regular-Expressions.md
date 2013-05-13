## Regular Expressions Interpolation in Pattern Matching 

With introduction of *string interpolation* in Scala 2.10 we finally got feature, that many modern languages already have. What made me extremely exited though, is the fact, how good this feature was integrated into the language and how customizable it is. In this post I would like to show you some examples of it. I will not concentrate on explaining how exactly it works, but instead I will show you one very cool application of it, which I found recently. It combines string interpolation with regular expressions. What is especially interesting is the way you can use it in the pattern matching expressions. 

It is something that I wanted to be able to make for a long time and actually [found one way to implement with type Dynamic](/post/49051516694/introduction-to-type-dynamic). That solution was a little bit crazy and looked like something that you normally will never use in real projects. Now I'm happy to show you solution that is actually looks nice (at least for regular expressions). I also want to notice that things I would like to share with you are shamelessly stolen from (were inspired by) this [video introduction to Scala 2.10 features](http://marakana.com/s/post/1461/what_is_new_in_scala_2_10_adriaan_moors_video) by [Adriaan Moors](https://twitter.com/adriaanm) and [this answer at StackOoverflow](http://stackoverflow.com/a/16256935/576766) by [sschaef](http://stackoverflow.com/users/465304/sschaef). I want to thank authors for giving me inspiration and educating me :)

<!-- more -->

### Adding String Interpolation to Regular Expressions

You probably already know how to create and use regexps in older versions
 of Scala:

    val FromToPattern = """(\d+) - (\d+)""".r

    val FromToPattern(from, to) = "100 - 200"

    println(s"From: $from, To: $to")

	// From: 100, To: 200

This code generally defines a regexp pattern and uses it in pattern match.

In Scala 2.10 we now have `StringContext` which compiler creates for interpolated strings. And you can actually enrich it with new methods, that can interpret interpolated string differently. That's exactly what I will implement for regular expressions by using *implicit class*:

    import util.matching.Regex
  
    implicit class RegexContext(sc: StringContext) {
      def r = new Regex(sc.parts.mkString, sc.parts.tail.map(_ => "x"): _*)
    }

It's all you need in order to be able to make pattern matching on regular expressions directly in `match` - `case`:

    "Amount is 100 USD" match {
      case r"Amount is (\d+)$amount ([A-Z]{3})$currency" =>
        println(s"Amount: $amount, Currency: $currency")
    }

	// Amount: 100, Currency: USD

As you can see I even able to save captured groups in the variables `amount` and `currency`.

### Combining Extractors within Regular Expression

Now I would like to show you more complicated example. This time I would like to save both amount and currency in `Money` object:

    case class Money(amount: Int, currency: String)

In order extract it from the string, I will define custom extractor for it, that generally parses string and extracts both values:

    object Money {
      def unapply(str: String) = str match {
        case r"(\d+)$amount\s+([A-Z]{3})$currency" =>
          Some(Money(amount.toInt, currency))
        case _ => None
      }
    }

With these definitions I'm able to "chain" these extractors together directly in regexp like this:

    "Amount is 100  USD" match {
      case r"Amount is (.+)${Money(money)}" =>
        println(money.copy(amount = money.amount + 15))
    }

	// Money(115,USD)

### Regexp Pattern Matching in val Definition

As you can imagine, you can also apply the same mechanism in all places, where you can use pattern matching. This includes `val` definitions. Let's rewrite our first example:

    object Int {
      def unapply(str: String) = Try(str.toInt).toOption
    }
  
    val r"(\d+)${Int(from)} - (\d+)${Int(to)}" = "45 - 123"
  
    println(s"from: $from, to: $to")

	// from: 45, to: 123