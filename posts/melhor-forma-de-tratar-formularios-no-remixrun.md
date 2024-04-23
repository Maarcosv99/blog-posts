Uma boa forma de utilizar formulário no Remix, é utilizando o pacote `remix-hook-form` . Que utiliza bases do pacote `react-hook-form` considerada a melhor para tratamento de formulário.

**Link para a documentação do pacote `remix-hook-form`:**
https://github.com/Code-Forge-Net/remix-hook-form

Como o formulário precisar ser tratado tanto no lado do servidor quanto no lado do cliente, o autor Alem Tuzlak criou esta biblioteca.

Eu testei ela e achei muito boa. Antes de testar eu estava planejando criar meu próprio pacote para tratamento de formulário utilizando qualquer tipo de validador.

No momento em que estava procurando mais informações, lembrei do pacote `react-hook-form` e acabei me deparando com o pacote `remix-hook-form` . Minha experiência, foi a melhor.

Antes de tudo, você precisa instalar os seguintes pacotes:

```shell
pnpm add remix-hook-form react-hook-form @hookform/resolvers zod
```

Abaixo esta uma página que trata múltiplos formulários e faz validação tanto no lado do cliente, como no lado do servidor.

Utilizei `intent` para diferenciar os formulários em uma única rota. Utilizei o componente `Form` e também utilizar `fetcher` .

```tsx
import type { ActionFunctionArgs, MetaFunction } from "@remix-run/node";
import { json } from "@remix-run/node"
import { Form, useFetcher, useNavigation, isRouteErrorResponse, useRouteError } from "@remix-run/react"

import { useRemixForm, getValidatedFormData, parseFormData } from "remix-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"

import { Label, Input, Button, Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@workspace/ui"
import { Loader2Icon } from "lucide-react"

export const meta: MetaFunction = () => [{ title: "Summary" }];

export default function Page() {
	const fetcher = useFetcher()
	const navigation = useNavigation()

	const loginForm = useRemixForm<LoginFormData>({ 
		mode: "onSubmit",
		resolver: loginResolver,
		submitData: { intent: "login" },
		fetcher
	})

	const signUpForm = useRemixForm<SignUpFormData>({
		mode: "onSubmit",
		resolver: signUpResolver,
		submitData: { intent: "sign-up" },
	})

  return (
		<div className="w-full min-h-dvh flex items-center justify-center">
			<div className="max-w-sm space-y-5">
				<fetcher.Form onSubmit={loginForm.handleSubmit}>
					<Card className="w-full">
						<CardHeader>
							<CardTitle className="text-2xl">Login</CardTitle>
							<CardDescription>
								Enter your email below to login to your account.
							</CardDescription>
						</CardHeader>
						<CardContent className="grid gap-4">
							<div className="grid gap-2">
								<Label htmlFor="email">Email</Label>
								<Input id="email" type="email" placeholder="m@example.com" {...loginForm.register("email")} />
								{loginForm.formState.errors.email && (
									<p className="text-xs text-red-500 font-medium">{loginForm.formState.errors.email.message}</p>
								)}
							</div>
							<div className="grid gap-2">
								<Label htmlFor="password">Password</Label>
								<Input id="password" type="password" {...loginForm.register("password")}/>
								{loginForm.formState.errors.password && (
									<p className="text-xs text-red-500 font-medium">{loginForm.formState.errors.password.message}</p>
								)}
							</div>
						</CardContent>
						<CardFooter>
							<Button type="submit" className="w-full">
								{(fetcher.formData?.get("intent") === '"login"')
									? <Loader2Icon className="w-4 h-4 animate-spin" />
									: "Sign In"
								}
							</Button>
						</CardFooter>
					</Card>
				</fetcher.Form>

				<Form onSubmit={signUpForm.handleSubmit}>
					<Card className="w-full">
						<CardHeader>
							<CardTitle className="text-2xl">SignUp</CardTitle>
							<CardDescription>
								Enter your email below to create your account.
							</CardDescription>
						</CardHeader>
						<CardContent className="grid gap-4">
							<div className="grid gap-2">
								<Label htmlFor="email">Email</Label>
								<Input id="email" type="email" placeholder="m@example.com" {...signUpForm.register("email")} />
								{signUpForm.formState.errors.email && (
									<p className="text-xs text-red-500 font-medium">{signUpForm.formState.errors.email.message}</p>
								)}
							</div>
						</CardContent>
						<CardFooter>
							<Button type="submit" className="w-full">
								{(navigation.formData?.get("intent") === '"sign-up"')
									? <Loader2Icon className="w-4 h-4 animate-spin" />
									: "Sign up"
								}
							</Button>
						</CardFooter>
					</Card>
				</Form>
			</div>
		</div>
  )
}

export function ErrorBoundary() {
	const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>
          {error.status} {error.statusText}
        </h1>
        <p>{error.data}</p>
      </div>
    );
  } else if (error instanceof Error) {
    return (
      <div>
        <h1>Error</h1>
        <p>{error.message}</p>
        <p>The stack trace is:</p>
        <pre>{error.stack}</pre>
      </div>
    );
  } else {
    return <h1>Unknown Error</h1>
	}
}

export const action = async ({ request }: ActionFunctionArgs) => {
	const formData = await parseFormData<{ intent?: string }>(request.clone())
	if (!formData.intent) throw json({ error: "Intent not found" }, { status: 404 })
	switch (formData.intent) {
		case 'sign-up': return await handleSignUp(request)
		case 'login': return await handleLogin(request)
		default: throw json({ error: "Invalid intent" }, { status: 404 })
	}
}

const loginSchema = z.object({ email: z.string().email(), password: z.string().min(8) })
const loginResolver = zodResolver(loginSchema)
type LoginFormData = z.infer<typeof loginSchema>
async function handleLogin(request: Request) {
	const { errors, data, receivedValues: defaultValues } =
		await getValidatedFormData<LoginFormData>(request, loginResolver);
	if (errors) return json({ errors, defaultValues })
	await new Promise(resolve => setTimeout(resolve, 1500))
	return json(data)
}

const signUpSchema = z.object({ email: z.string().email() })
const signUpResolver = zodResolver(signUpSchema)
type SignUpFormData = z.infer<typeof signUpSchema>
async function handleSignUp(request: Request) {
	const { errors, data, receivedValues: defaultValues } =
		await getValidatedFormData<SignUpFormData>(request, signUpResolver);
	if (errors) return json({ errors, defaultValues })
	await new Promise(resolve => setTimeout(resolve, 1500))
	return json(data)
}

```
