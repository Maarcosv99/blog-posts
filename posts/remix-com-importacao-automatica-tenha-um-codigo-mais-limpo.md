Se você está utilizando `Vite` , o plugin `unimport` vai te ajudar bastante a deixar seus componentes mais limpos.

Antes de tudo, instale o pacote:
```bash
pnpm add unimport
```

Depois defina este plugins e estas configurações em seu arquivo `vite.config.ts` 

```ts
import { defineConfig } from "vite";
import Unimport from 'unimport/unplugin'

export default defineConfig({
  plugins: [
		Unimport.vite({
			dts: true,
			dirs: [
				'./app/components/*',
				'../../packages/ui/src/components/*'
			],
			presets: [
				'react',
				{
					from: '@remix-run/react',
					imports: [
						'useLoaderData', 'useActionData', 'useLocation', 'useNavigation',
						'useNavigate', 'useParams', 'useAsyncError', 'useAsyncValue',
						'useBeforeUnload', 'useBlocker', 'useFetcher', 'useFetchers',
						'useFormAction', 'useHref', 'useMatches', 'useNavigationType',
						'useOutlet', 'useOutletContext', 'unstable_usePrompt', 'useResolvedPath',
						'useRevalidator', 'useRouteError', 'useRouteLoaderData', 'useSearchParams',
						'useSubmit', 'unstable_useViewTransitionState', 'Link', 'Form', 'Await', 'Links',
						'Outlet', 'NavLink', 'PrefetchPageLinks'
					],
					priority: 10
				},
				{
					from: '@remix-run/node',
					imports: [
						'json', 'redirect', 'defer', 'createCookie', 'isRouteErrorResponse', 'redirectDocument',
						'createCookieSessionStorage', 'createMemorySessionStorage', 'createFileSessionStorage'
					]
				},
				{
					from: '@remix-run/node',
					type: true,
					imports: [
						'ActionFunctionArgs', 'LoaderFunctionArgs',
						'MetaFunctionArgs', 'MetaFunction', 'ClientActionFunctionArgs',
						'ClientLoaderFunctionArgs', 'HeadersFunction', 'LinksFunction',
						'ShouldRevalidateFunction'
					]
				}
			]	
		})
	],
});

```

Após instalar o pacote e configurar o plugin, você pode criar componentes como este abaixo, sem utilizar nenhuma importação.

> A importação será necessária apenas para casos independentes.

```tsx
export const meta: MetaFunction = () => {
  return [{ title: "Summary" }];
};

export const action = async ({ request }: ActionFunctionArgs) => {
	const formData = await request.clone().formData()
	return json({ success: true, message: formData.get("name")?.toString() })
}

export const loader = ({ request }: LoaderFunctionArgs) => {
	return json({ hello: 'world' })
}

export default function Page() {
	const loaderData = useLoaderData<typeof loader>()
	const actionData = useActionData<typeof action>()

  return (
    <main className="w-full min-h-screen flex items-center justify-center bg-zinc-100">
      <div className="sm:w-[500px] space-y-3">
				<Link to="eae">
					<Headline name="Teste" />
				</Link>
				{JSON.stringify(loaderData)}
				
				<Card>
					<CardHeader>
						<CardTitle>Testing</CardTitle>
						{actionData ? <p>{actionData.message}</p> : null}
					</CardHeader>
					<CardContent>
						<Form method="post" className="space-y-2">
							<Label htmlFor="name">Name</Label>
							<Input name="name" id="name" type="text" />
							<Button type="submit" variant="default">
								Add
							</Button>
						</Form>
					</CardContent>
				</Card>
			</div>
    </main>
  );
}
```
