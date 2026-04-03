# Trade Intent Model (TIM) as a Domain-Specific Language for Precise Cross-Venue Trade Execution Planning

Bill Sun

## Abstract

Most AI trading systems still use the wrong abstraction boundary. A user expresses what they want in natural language, but the system often tries to jump directly from that ambiguous description to executor-specific API calls. The result is brittle tooling, opaque failures, and duplicated integration logic across venues. TIM, the Trading Intent Model, proposes a different decomposition. It treats trading as a compilation pipeline: natural language is translated into an unambiguous intermediate representation, validated against a trade-intent DSL, and then lowered into layered execution policies and venue-specific instructions. The key contribution is not a new trading interface. It is a verifiable trade intent DSL that can describe orders across venues, asset classes, and execution backends while preserving a clean boundary between planning and execution.

## 1. Introduction

The central problem in AI trading infrastructure is ambiguity management. Human users naturally describe trades at a high level: what they want to own, what constraints they care about, which venues they trust, and what combinations of positions they want to assemble. Execution systems, by contrast, require a low-level specification: product identifiers, chain or venue routing, leg structure, price and quantity fields, settlement assumptions, and risk controls. When these two layers are connected directly, the result is a black box. A failed trade may reflect a misunderstanding of the user request, an incorrect intermediate representation, or a bad venue adapter, yet the system offers no principled way to separate these failure modes.

TIM is built around a stronger claim: trading should be treated as compilation. Natural language is the source language. A trade intent is the intermediate representation. Layered execution plans and venue-specific instructions are the target code. This changes the engineering objective. Instead of designing one more trading interface, TIM defines a domain-specific language for trade intent: a precise, verifiable, cross-venue schema that captures what the user wants in a form that machines can validate and downstream systems can compile further.

The strongest one-sentence description is therefore the following:

`TIM is not merely a trading interface; it is a compiler that translates natural-language trading intent into an unambiguous, verifiable, and cross-venue executable trade intent DSL.`

This framing shifts the value proposition from convenience to correctness. Every stage of translation becomes inspectable, testable, and debuggable.

## 2. From Natural Language to Trade Intent

The first layer of the system is semantic parsing for trading. This is analogous to how modern search systems map user utterances into structured query representations, or how systems such as CVXPY allow a user to specify optimization problems in a high-level language that is then compiled into a canonical form. In the same spirit, TIM begins with human intent expressed in natural language and compiles it into a formal intermediate representation.

Consider the following trade vector:

`buy $10 of NVDAx on SOL, $5 Pax Gold on the best ETH L2 by liquidity, buy one YES contract for "can NVDA pass $200 in Feb 2026", buy one "2 times" contract for "number of Fed cuts this year", and place an AAPL June 6 bull call spread with strikes 190 and 200 on Alpaca`

This sentence is meaningful to a human, but it is not yet executable. It contains implicit decomposition, venue selection logic, product-family changes, and multiple asset classes. The sentence does not say whether it should be executed atomically, whether the venue-selection rule is resolved at compile time or execution time, or how the options spread should be represented internally. The purpose of the first layer is therefore not execution. It is disambiguation.

The output of that layer is a trade intent: a formal object that preserves the user’s economic meaning while eliminating linguistic ambiguity. The concrete syntax may be XML, JSON, or an in-memory object. That choice is secondary. The essential property is mathematical precision. A trade intent must be structured, typed, and mechanically checkable.

## 3. Trade Intent as a Domain-Specific Language

The second layer is the intent language itself. TIM treats trade intent as a domain-specific language for trading. Like a good mathematical language, it must satisfy four requirements.

First, it must be unambiguous. A valid intent should mean one thing.

Second, it must be verifiable. Required fields, admissible value ranges, disjoint alternatives, and structural invariants must be checkable before execution.

Third, it must be compositional. A complex trade vector should decompose into explicit intent objects or structured multi-leg intents rather than disappearing into an opaque executor call.

Fourth, it must be venue-agnostic at the semantic layer. The user’s request should be representable before the system commits to Hyperliquid, Polymarket, Solana, Ethereum, Alpaca, Interactive Brokers, Robinhood, or a prime-broker workflow.

Fifth, it must be execution-aware without collapsing into low-level execution. A high-level intent should be precise enough to support cross-venue routing, liquidity aggregation, and downstream decomposition while still remaining above the level of TWAP schedules, AMM splitting logic, or order-book tactics.

This is why TIM uses schema-defined intent types. In the current implementation, the schema is the source of truth for structure, validation, templates, and dispatch metadata. The schema does not merely document the system. It defines the language that the system accepts. In that sense, TIM is closer to a compiler front end than to an ordinary API wrapper.

## 4. Layered Compilation for Trading

The full pipeline can be stated in four deterministic layers.

1. Natural language -> high-level trade intent.
2. High-level trade intent -> validated trade intent object and execution task graph.
3. Execution task graph -> venue-specific execution instructions.
4. Venue-specific execution instructions -> final execution.

Each translation increases determinacy. This is the architectural point.

If a failure occurs, the system no longer reports only that a trade failed. It can localize the failure. The natural-language parser may have misunderstood the request. The intent constructor may have produced an invalid object. The execution planner may have decomposed the high-level objective incorrectly. The execution adapter may have translated the validated intent incorrectly for a particular venue. By separating these stages, TIM turns a black-box failure into a debuggable systems problem.

This is also the reason the schema must sit in the middle of the stack. If the intermediate representation is informal, the entire compiler story collapses. If it is precise, then every downstream transformation can be validated against a known contract.

## 5. High-Level Planning versus Low-Level Execution

This layered view is standard in robotics and should be standard in trading. A robot has high-level planning and low-level motor execution. Trading systems have the same structural separation. A user states what position they want, by when, under what constraints, and across which candidate venues. That is a high-level planning problem. How to realize that objective in a particular microstructure is a different problem.

The useful high-level object is therefore not an order ticket. It is a high-level trade intent: an unambiguous, cross-venue, liquidity-aware representation of the user objective. Once that object exists, TIM can decompose it into lower-level tasks. Those tasks may then be executed by deterministic execution algorithms such as TWAP or VWAP, by naive on-chain slicing across AMMs, or by richer venue-specific logic that adapts to market state.

Consider the instruction `buy $10 million of AAPL by end of day` or `buy $10 million of AAPL by end of week`. The high-level intent captures the asset, size, time horizon, constraints, and optimization objective. The low-level execution layer then decides how to realize that intent. The system may choose a TWAP schedule, a VWAP schedule, block liquidity sourcing, exchange-specific slicing, or another deterministic execution policy. In on-chain settings, it may decompose the trade into route discovery, AMM splitting, and liquidity-sensitive order placement across venues. These are not different user intents. They are different compilations of the same intent into low-level tasks.

This decomposition also creates a principled place for microstructure-level improvement. A base execution algorithm can be blended with low-latency execution logic, order-book microstructure alpha, or other short-horizon tactics that improve implementation quality. In other words, the same high-level intent can admit multiple low-level realizations with different execution quality. That is exactly why the layers must be separated.

## 6. Cross-Venue Unification

The third major claim of TIM is that one intent schema should be able to describe trading across markets, venues, and asset classes. This does not mean every venue becomes identical. It means the semantic layer is unified even when the execution layer remains heterogeneous.

That distinction is crucial. A prediction market contract on Polymarket, a spot swap on Solana, a perpetual trade on Hyperliquid, and a multi-leg options spread on Alpaca should not be forced into the same executor API. They should instead be representable inside the same intent language. Once that language exists, the upper layers of the system can reason uniformly about validation, orchestration, batching, risk checks, and failure attribution.

TIM therefore plays a role analogous to a LEAN-style domain-specific language for trading, but with the additional requirement that the representation be explicitly machine-verifiable and suitable for multi-stage compilation. Users specify what they want. TIM compiles that request into a standardized trade intent representation. Execution planners decompose that representation into tasks. Venue adapters then compile those tasks into concrete execution instructions for the destination system.

## 7. TIM as Implemented

The current TIM repository already reflects this compiler interpretation. Intent schemas live as YAML files. The agent-facing template surface is generated from those schemas. A generic XML parser converts submitted intent payloads into JSON without embedding product-specific logic. A schema validator checks the resulting value for structural correctness. A dispatcher routes the validated intent to an executor according to configured rules such as intent type and chain identifier. Conceptually, the next layer is an execution planner that decomposes high-level intent into venue-specific tasks before final execution.

This design is intentionally narrow. TIM does not attempt to be the execution engine for every market. It acts as the contract layer between planning and execution. New intent types are introduced primarily by adding schema files. New routes are introduced primarily by configuration.

## 8. Conclusion

The right way to think about TIM is not as a thin trading frontend. It is a compiler boundary for trading systems. Natural language belongs at the top, because that is how humans express goals. Low-level execution policies and venue-specific instructions belong at the bottom, because that is how markets are actually accessed. Between them there must exist a precise intermediate language that removes ambiguity, supports validation, and composes across venues. If AI agents are going to participate in financial execution, the infrastructure must standardize meaning before it standardizes action. Trade intent is that meaning layer.
