# 🦜️🔗 LangGraphGo

[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/tmc/langgraphgo)


## Quick Start


This is a simple example of how to use the library to create a simple chatbot that uses OpenAI to generate responses.

```go
import (
	"context"
	"fmt"

	"github.com/tmc/langchaingo/llms"
	"github.com/tmc/langchaingo/llms/openai"
	"github.com/tmc/langgraphgo/graph"
)

func main() {
	model, err := openai.New()
	if err != nil {
		panic(err)
	}

	g := graph.NewMessageGraph()

	g.AddNode("oracle", func(ctx context.Context, state []llms.MessageContent) ([]llms.MessageContent, error) {
		r, err := model.GenerateContent(ctx, state, llms.WithTemperature(0.0))
		if err != nil {
			return nil, err
		}
		return append(state,
			llms.TextParts(llms.ChatMessageTypeAI, r.Choices[0].Content),
		), nil

	})
	g.AddNode(graph.END, func(ctx context.Context, state []llms.MessageContent) ([]llms.MessageContent, error) {
		return state, nil
	})

	g.AddEdge("oracle", graph.END)
	g.SetEntryPoint("oracle")

	runnable, err := g.Compile()
	if err != nil {
		panic(err)
	}

	ctx := context.Background()
	// Let's run it!
	res, err := runnable.Invoke(ctx, []llms.MessageContent{
		llms.TextParts(llms.ChatMessageTypeHuman, "What is 1 + 1?"),
	})
	if err != nil {
		panic(err)
	}

	fmt.Println(res)

	// Output:
	// [{human [{What is 1 + 1?}]} {ai [{1 + 1 equals 2.}]}]
}
```
