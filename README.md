See this video:
https://youtu.be/NeQYfp7h_pQ

Then in the app.blade.php file do below:
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}" @class(['dark' => ($appearance ?? 'system') === 'dark'])>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="csrf-token" content="{{ csrf_token() }}">

        {{-- Inline script to detect system dark mode preference and apply it immediately --}}
        <script>
            (function() {
                const appearance = '{{ $appearance ?? "system" }}';
                
                // Store appearance preference in localStorage
                const storedAppearance = localStorage.getItem('appearance');
                const effectiveAppearance = storedAppearance || appearance;

                if (effectiveAppearance === 'system') {
                    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
                    if (prefersDark) {
                        document.documentElement.classList.add('dark');
                    } else {
                        document.documentElement.classList.remove('dark');
                    }
                    
                    // Listen for system theme changes
                    window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', e => {
                        if (effectiveAppearance === 'system') {
                            if (e.matches) {
                                document.documentElement.classList.add('dark');
                            } else {
                                document.documentElement.classList.remove('dark');
                            }
                        }
                    });
                } else if (effectiveAppearance === 'dark') {
                    document.documentElement.classList.add('dark');
                } else {
                    document.documentElement.classList.remove('dark');
                }
            })();
        </script>

        {{-- Inline style to set the HTML background color based on our theme --}}
        <style>
            :root {
                --background: oklch(1 0 0);
                --foreground: oklch(0.145 0 0);
            }
            
            .dark {
                --background: oklch(0.145 0 0);
                --foreground: oklch(0.985 0 0);
            }

            html {
                background-color: var(--background);
                color: var(--foreground);
            }

            body {
                margin: 0;
                min-height: 100vh;
            }
        </style>

        <title inertia>{{ config('app.name', 'Laravel') }}</title>

        <link rel="icon" href="/favicon.ico" sizes="any">
        <link rel="icon" href="/favicon.svg" type="image/svg+xml">
        <link rel="apple-touch-icon" href="/apple-touch-icon.png">
        <link rel="manifest" href="/site.webmanifest">

        <link rel="preconnect" href="https://fonts.bunny.net">
        <link href="https://fonts.bunny.net/css?family=instrument-sans:400,500,600" rel="stylesheet" />

        {{-- For production/live server, use built assets --}}
        @if (app()->isProduction())
            @php
                // Check for manifest in build/.vite/ directory (standard Vite location)
                $manifestPath = public_path('build/.vite/manifest.json');
                
                // Fallback to root build directory if not found
                if (!file_exists($manifestPath)) {
                    $manifestPath = public_path('build/manifest.json');
                }
                
                // Check if manifest exists before trying to read it
                if (file_exists($manifestPath)) {
                    $manifest = json_decode(file_get_contents($manifestPath), true);
                } else {
                    $manifest = null;
                }
            @endphp
            
            @if ($manifest && isset($manifest['resources/js/app.ts']))
                <script type="module" src="{{ asset('build/' . $manifest['resources/js/app.ts']['file']) }}" defer></script>
                @if (isset($manifest['resources/js/app.ts']['css']))
                    @foreach ($manifest['resources/js/app.ts']['css'] as $css)
                        <link rel="stylesheet" href="{{ asset('build/' . $css) }}">
                    @endforeach
                @endif
            @else
                {{-- Fallback to Vite if manifest not found or invalid --}}
                @vite(['resources/js/app.ts'])
            @endif
        @else
            {{-- For development --}}
            @vite(['resources/js/app.ts', "resources/js/pages/{$page['component']}.vue"])
        @endif

        @inertiaHead
        @routes
    </head>
    <body class="font-sans antialiased">
        @inertia
        
        {{-- Script to handle dark mode persistence --}}
        <script>
            // Listen for appearance changes from your Vue components
            window.addEventListener('appearance-changed', (event) => {
                const { appearance } = event.detail;
                localStorage.setItem('appearance', appearance);
                
                if (appearance === 'system') {
                    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
                    if (prefersDark) {
                        document.documentElement.classList.add('dark');
                    } else {
                        document.documentElement.classList.remove('dark');
                    }
                } else if (appearance === 'dark') {
                    document.documentElement.classList.add('dark');
                } else {
                    document.documentElement.classList.remove('dark');
                }
            });

            // Initialize with stored preference on page load
            document.addEventListener('DOMContentLoaded', function() {
                const storedAppearance = localStorage.getItem('appearance');
                if (storedAppearance && storedAppearance !== '{{ $appearance ?? "system" }}') {
                    // Trigger event to sync with Vue components
                    window.dispatchEvent(new CustomEvent('appearance-changed', {
                        detail: { appearance: storedAppearance }
                    }));
                }
            });
        </script>
    </body>
</html>
