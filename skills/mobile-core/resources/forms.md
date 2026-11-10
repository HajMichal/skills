# Forms & Keyboard Handling

## Keyboard Avoiding View

```tsx
import { KeyboardAvoidingView, Platform, ScrollView } from 'react-native';

export function LoginScreen() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      className="flex-1"
    >
      <ScrollView
        contentContainerClassName="grow justify-center px-6"
        keyboardShouldPersistTaps="handled"
      >
        <Input
          placeholder={t('login.email.placeholder')}
          keyboardType="email-address"
          autoCapitalize="none"
          autoComplete="email"
          returnKeyType="next"
        />
        <Input
          placeholder={t('login.password.placeholder')}
          secureTextEntry
          autoComplete="password"
          returnKeyType="done"
          onSubmitEditing={handleSubmit}
        />
      </ScrollView>
    </KeyboardAvoidingView>
  );
}
```

## Input Types Cheatsheet

| Input type | `keyboardType` | `autoCapitalize` | `autoComplete` |
|-----------|---------------|-----------------|---------------|
| Email | `email-address` | `none` | `email` |
| Phone | `phone-pad` | `none` | `tel` |
| Password | (default) | `none` | `password` / `new-password` |
| Number | `numeric` | `none` | `off` |
| Name | (default) | `words` | `name` |

## Form Validation

```tsx
// Validate with Zod on submit
const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password too short'),
});

function handleSubmit() {
  const result = loginSchema.safeParse({ email, password });
  if (!result.success) {
    setErrors(result.error.flatten().fieldErrors);
    return;
  }
  mutate(result.data);
}
```

## Dismiss Keyboard

```tsx
import { Keyboard, Pressable } from 'react-native';

// Dismiss on tap outside
<Pressable onPress={Keyboard.dismiss} className="flex-1">
  {/* form content */}
</Pressable>
```
