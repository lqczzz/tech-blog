---
title: break 和 continue
date: 2018-10-26 18:49:42
tags: golang思考
---

golang的break和continue挺好用的，和别的语言不太一样

# break
golang的`break`关键字`for`,`switch`,`select`会跳出三个关键字的包裹

>[A "break" statement terminates execution of the innermost "for", "switch", or "select" statement within the same function. —— 《The Go Programming Language Specification》](https://golang.org/ref/spec#Break_statements)

下面一段代码

    for i := 0; i < 6; i++ {
		switch i {
		case 2:
			break
		default:
			fmt.Println(i)
		}
	}
    // go run main.go：
    // 0
    // 1
    // 3
    // 4
    // 5

如果想跳出更上一层的`for`关键字，需要指定`label`

    forLoop:
        for i := 0; i < 6; i++ {
            switch i {
            case 2:
                break forLoop
            default:
                fmt.Println(i)
            }
        }

    // go run main.go:
    // 0
    // 1


# continue

`continue`也可以指定label

    forLoop:
        for i := 0; i < 6; i++ {
            switch i {
            case 2:
                continue forLoop
            default:
                fmt.Println(i)
            }
        }
    // go run main.go
    // 0
    // 1
    // 3
    // 4
    // 5



