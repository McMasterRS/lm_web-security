---
layout: default
title: Adding SSO to a Dockerized Next.js Application
parent: Authentication
nav_order: 2
---

# Adding SSO to a Dockerized Next.js Application

![azure-nextjs](assets/img/azure-nextjs.png)

_Image retrieved from [medium.com](https://medium.com/codex/running-next-js-on-azure-app-services-84f707af761d)_  

Modern websites often require an authentication solution to store user information and limit access to certain features and resources. Developing such a solution from scratch involves a lot of time and resources in addition to the numerous challenges such as handling security and data storage. Luckily, there are existing solutions that help developers with the authentication process such as the Microsoft Azure Directory (AD) single sign-on. Single sign-on (SSO) is an authentication method that allows users to sign in using one set of credentials to multiple independent software systems. Using SSO means users will not have to create new credentials for every application they use. With SSO, users can access all needed applications with a single account, which reduces the risk of users using repeated passwords while saving them the hassle of creating and remembering a new pair of credentials. Microsoft Azure relies on the IDC (OpenID Connect) and OAuth 2.0 industry standard protocols to support authentication and authorization into various application. McMaster University uses SSO to provide authentication and authorization services using MacIDs for its web applications. The University Technology Services (UTS) manage the McMaster Azure Directory and assist developers with setting up new applications for SSO. 

We will now cover the process of adding single sign-on to a dockerized Next.js application. 

## Install identity and MUI packages
The azure identity related `npm` packages must be installed in the project to enable user authentication. We will also make use of the Material UI (MUI) library for styling and components. 

Add the `msal-react` and `msal-browser` packages to your project using the following command:  

```bash
npm install @azure/msal-browser @azure/msal-react
```

Add the MUI library to your project:  

```bash
npm install @mui/material @emotion/react @emotion/styled
```

## Creating the authentication configuration file

1. In the `client` folder of your project, create a new directory called `config`. Create a new file called `authConfig.ts` inside this directory.
2. Open `authConfig.ts` and add the following code snippet:  

```ts
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License.
 */

import { LogLevel } from "@azure/msal-browser";
import * as process from "process";

/**
 * Configuration object to be passed to MSAL instance on creation.
 * For a full list of MSAL.js configuration parameters, visit:
 * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md
 */

export const msalConfig = {
    auth: {
        clientId: `${process.env.CLIENT_ID}`,
        authority: `https://login.microsoftonline.com/${process.env.TENANT_ID}`,
        redirectUri: `${process.env.REDIRECT_URI}`,
    },
    cache: {
        cacheLocation: "sessionStorage", // This configures where your cache will be stored
        storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    },
    system: {
        loggerOptions: {
            loggerCallback: (level: LogLevel, message: string, containsPii: boolean) => {
                if (containsPii) {
                    return;
                }
                switch (level) {
                    case LogLevel.Error:
                        console.error(message);
                        return;
                    case LogLevel.Info:
                        console.info(message);
                        return;
                    case LogLevel.Verbose:
                        console.debug(message);
                        return;
                    case LogLevel.Warning:
                        console.warn(message);
                        return;
                    default:
                        return;
                }
            }
        }
    }
};

/**
 * Scopes you add here will be prompted for user consent during sign-in.
 * By default, MSAL.js will add OIDC scopes (openid, profile, email) to any login request.
 * For more information about OIDC scopes, visit:
 * https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-permissions-and-consent#openid-connect-scopes
 */
export const loginRequest = {
    scopes: ["User.Read"],
};

/**
 * Add here the scopes to request when obtaining an access token for MS Graph API. For more information, see:
 * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/resources-and-scopes.md
 */
export const graphConfig = {
    graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
};
```

3. Create an  `.env.development.local` file in the `client` directory
4. Add the following definitions to `.env.development.local`:  

```
CLIENT_ID=Enter_the_App_Id_Here
TENANT_ID=Enter_the_Tenant_Id_here
REDIRECT_URI=Enter_Redirect_Uri_here
```

5. Replace the following values with the values from the app registration file you received from UTS:
  - `CLIENT_ID` - The identifier of the application, also referred to as the client. Replace `Enter_the_Application_Id_Here` with the **App-Id** value provided by UTS.
  - `TENANT_ID` - the identifier of the tenant where the application is registered. Replace  `Enter_the_Tenant_Id_Here` with the **Tenant Id (McMaster)** provided by UTS.
  - `REDIRECT_URI`- the redirect URI provided to UTS in the registration ticket. Replace `Enter_the_Redirect_Uri_Here` with the redirect URI you provided to UTS.

## Modify `template.tsx` to include the authentication provider
Open the `app/template.tsx` file and replace the contents of the file with the following code snippet to use the `msal` packages:  

```ts
'use client';

import '../styles/globals.css'
import {PageLayout} from '../Components/Layout/PageLayout';
import React from 'react';
import {PublicClientApplication} from '@azure/msal-browser';
import {MsalProvider} from '@azure/msal-react';
import {msalConfig} from '../config/authConfig';
import {createTheme, ThemeProvider} from '@mui/material/styles'
import {CssBaseline} from "@mui/material";
import useMediaQuery from '@mui/material/useMediaQuery'

const msalInstance = new PublicClientApplication(msalConfig);

export default function Template({children}: {children?: React.ReactNode} ) {
    const prefersDarkMode = useMediaQuery('(prefers-color-scheme: dark)')

    const theme = createTheme({
        palette: {
            mode: prefersDarkMode ? "dark" : "light"
        }
    });
    return (
        <ThemeProvider theme={theme}>
            <CssBaseline />
            <MsalProvider instance={msalInstance}>
                <PageLayout>
                    <center>
                        {children}
                    </center>
                </PageLayout>
            </MsalProvider>
        </ThemeProvider>
    )
}
```

Make sure that the `msalInstance` constant is always declared outside the body of the `Template` function to avoid any unpredictable behavior caused by race conditions. We added some boilerplate code to handle switching to dark mode if the user has dark mode enabled on their browser or OS settings.

## Add components to the application

The project needs extra files to be created in order to render the the page layout, display the user profile data, and handle the sign in and sign out workflows.  

In the `client` directory of your project, create a `Components` directory with an `Authentication` subdirectory inside of it.  

Create a new file inside the `Authentication` directory called `ProfileData.tsx` and add the following code to it:  

```ts
import React from "react";
/**
 * Renders information about the user obtained from MS Graph
 * @param props
 */
interface  graphData {
    givenName: string
    surname: string
    userPrincipalName: string
    id: number
}

interface ProfileDataProps {
    graphData: graphData
}

export const ProfileData = (props: ProfileDataProps) => {
    return (
        <div id="profile-div">
            <p>
                <strong>First Name: </strong> {props.graphData.givenName}
            </p>
            <p>
                <strong>Last Name: </strong> {props.graphData.surname}
            </p>
            <p>
                <strong>Email: </strong> {props.graphData.userPrincipalName}
            </p>
            <p>
                <strong>Id: </strong> {props.graphData.id}
            </p>
        </div>
    );
};
```

The `ProfileData` component is used to display the user information (i.e., first name, last name, email and ID) after a user has successfully logged in.  

We will now create the `SignInButton` and `SignOutButton` components. Create a new file called `SignInButton.tsx` inside the `Authencation` directory and add the following content to it:  

```ts
{% raw %}
import React from "react";
import { useMsal } from "@azure/msal-react";
import { loginRequest } from "../../config/authConfig";
import Button from '@mui/material/Button';
import ClickAwayListener from '@mui/material/ClickAwayListener';
import Grow from '@mui/material/Grow';
import Paper from '@mui/material/Paper';
import Popper from '@mui/material/Popper';
import MenuItem from '@mui/material/MenuItem';
import MenuList from '@mui/material/MenuList';
/**
 * Renders a drop down button with child buttons for logging in with a popup or redirect
 * Note the [useMsal] package
 */

export const SignInButton = () => {
    const { instance } = useMsal();

    const handleLogin = (loginType: string) => {
        if (loginType === "popup") {
            instance.loginPopup(loginRequest).catch((e) => {
                console.log(e);
            });
        } else if (loginType === "redirect") {
            instance.loginRedirect(loginRequest).catch((e) => {
                console.log(e);
            });
        }
    };

    const [open, setOpen] = React.useState(false);
    const anchorRef = React.useRef<HTMLButtonElement>(null);

    const handleToggle = () => {
        setOpen((prevOpen) => !prevOpen);
    };

    const handleClose = (event: Event | React.SyntheticEvent) => {
        if (
            anchorRef.current &&
            anchorRef.current.contains(event.target as HTMLElement)
        ) {
            return;
        }

        setOpen(false);
    };

    function handleListKeyDown(event: React.KeyboardEvent) {
        if (event.key === 'Tab') {
            event.preventDefault();
            setOpen(false);
        } else if (event.key === 'Escape') {
            setOpen(false);
        }
    }

    // return focus to the button when we transitioned from !open -> open
    const prevOpen = React.useRef(open);
    React.useEffect(() => {
        if (prevOpen.current && !open) {
            anchorRef.current!.focus();
        }

        prevOpen.current = open;
    }, [open]);


    return (
        <div>
            <Button
                ref={anchorRef}
                id="composition-button"
                aria-controls={open ? 'composition-menu' : undefined}
                aria-expanded={open ? 'true' : undefined}
                aria-haspopup="true"
                onClick={handleToggle}
                sx={{my: 2, color: 'white', display: 'block'}}
            >
                Sign In
            </Button>
            <Popper
                open={open}
                anchorEl={anchorRef.current}
                role={undefined}
                placement="bottom-start"
                transition
                disablePortal
            >
                {({ TransitionProps, placement }) => (
                    <Grow
                        {...TransitionProps}
                        style={{
                            transformOrigin:
                                placement === 'bottom-start' ? 'left top' : 'left bottom',
                        }}
                    >
                        <Paper>
                            <ClickAwayListener onClickAway={handleClose}>
                                <MenuList
                                    autoFocusItem={open}
                                    id="composition-menu"
                                    aria-labelledby="composition-button"
                                    onKeyDown={handleListKeyDown}
                                >
                                    <MenuItem onClick={() => handleLogin("popup")}>Sign in using Popup</MenuItem>
                                    <MenuItem onClick={() => handleLogin("redirect")}>Sign in using Redirect</MenuItem>
                                </MenuList>
                            </ClickAwayListener>
                        </Paper>
                    </Grow>
                )}
            </Popper>
        </div>
    );
};
{% endraw %}
```

The `SignInButton` component renders a dropdown button with child buttons for logging in with a popup or with redirect.  

Similarly, create a `SignOutButton.tsx` file inside the `Authentication` directory and add the following code snippet to it:  

```ts
{% raw %}
import React from "react";
import { useMsal } from "@azure/msal-react";
import Button from "@mui/material/Button";
import Popper from "@mui/material/Popper";
import Grow from "@mui/material/Grow";
import Paper from "@mui/material/Paper";
import ClickAwayListener from "@mui/material/ClickAwayListener";
import MenuList from "@mui/material/MenuList";
import MenuItem from "@mui/material/MenuItem";

/**
 * Renders a sign out button
 */
export const SignOutButton = () => {
    const { instance } = useMsal();

    const handleLogout = (logoutType: string) => {
        if (logoutType === "popup") {
            instance.logoutPopup({
                postLogoutRedirectUri: "/",
                mainWindowRedirectUri: "/",
            });
        } else if (logoutType === "redirect") {
            instance.logoutRedirect({
                postLogoutRedirectUri: "/",
            });
        }
    };

    const [open, setOpen] = React.useState(false);
    const anchorRef = React.useRef<HTMLButtonElement>(null);

    const handleToggle = () => {
        setOpen((prevOpen) => !prevOpen);
    };

    const handleClose = (event: Event | React.SyntheticEvent) => {
        if (
            anchorRef.current &&
            anchorRef.current.contains(event.target as HTMLElement)
        ) {
            return;
        }

        setOpen(false);
    };

    function handleListKeyDown(event: React.KeyboardEvent) {
        if (event.key === 'Tab') {
            event.preventDefault();
            setOpen(false);
        } else if (event.key === 'Escape') {
            setOpen(false);
        }
    }

    // return focus to the button when we transitioned from !open -> open
    const prevOpen = React.useRef(open);
    React.useEffect(() => {
        if (prevOpen.current && !open) {
            anchorRef.current!.focus();
        }

        prevOpen.current = open;
    }, [open]);


    return (
        <div>
            <Button
                ref={anchorRef}
                id="composition-button"
                aria-controls={open ? 'composition-menu' : undefined}
                aria-expanded={open ? 'true' : undefined}
                aria-haspopup="true"
                onClick={handleToggle}
                sx={{my: 2, color: 'white', display: 'block'}}
            >
                Sign Out
            </Button>
            <Popper
                open={open}
                anchorEl={anchorRef.current}
                role={undefined}
                placement="bottom-start"
                transition
                disablePortal
            >
                {({ TransitionProps, placement }) => (
                    <Grow
                        {...TransitionProps}
                        style={{
                            transformOrigin:
                                placement === 'bottom-start' ? 'left top' : 'left bottom',
                        }}
                    >
                        <Paper>
                            <ClickAwayListener onClickAway={handleClose}>
                                <MenuList
                                    autoFocusItem={open}
                                    id="composition-menu"
                                    aria-labelledby="composition-button"
                                    onKeyDown={handleListKeyDown}
                                >
                                    <MenuItem onClick={() => handleLogout("popup")}>Sign out using Popup</MenuItem>
                                    <MenuItem onClick={() => handleLogout("redirect")}>Sign out using Redirect</MenuItem>
                                </MenuList>
                            </ClickAwayListener>
                        </Paper>
                    </Grow>
                )}
            </Popper>
        </div>
    );
};
{% endraw %}
```

Akin to the `SignInButton`, the `SignOutButton` component renders a dropdown button with two sign out options.  

## Create the `PermissionGate` component

We will create a `PermissionGate` component that prevents users from accessing our webpage before logging in. If a user is not yet successfully authenticated, the `PermissionGate` will display a `Modal` message informing the user that they need to login to access our website.

Create a new `PermissionGate` directory inside the components directory and add a `PermissionGate.tsx` file to it. Add the following code to `PermissionGate.tsx`:  

```ts
{% raw %}
import React from 'react'
import Box from '@mui/material/Box'
import Modal from '@mui/material/Modal'
import {AuthenticatedTemplate, UnauthenticatedTemplate, useIsAuthenticated, useMsal} from "@azure/msal-react";
import {loginRequest} from "../../config/authConfig";
import {InteractionStatus} from "@azure/msal-browser";

interface PermissionGateProps {
    children:
        | React.ReactElement<any, string | React.JSXElementConstructor<any>>
        | React.ReactPortal
}

export default function PermissionGate({children}: PermissionGateProps) {
    const isAuthenticated = useIsAuthenticated();
    const { instance, inProgress } = useMsal();

    return (
        <>
            <AuthenticatedTemplate>
                <>{children}</>
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <Modal
                    open
                    onClose={() => {
                        if (inProgress === InteractionStatus.None && !isAuthenticated) {
                            instance.loginRedirect(loginRequest);
                        }
                    }}
                >
                    <Box
                        sx={{
                            position: 'absolute' as 'absolute',
                            top: '50%',
                            left: '50%',
                            transform: 'translate(-50%, -50%)',
                            width: 400,
                            border: '2px solid #000',
                            bgcolor: 'background.paper',
                            color: 'black',
                            borderRadius: '16px',
                            boxShadow: 24,
                            p: 4,
                        }}
                    >
                        {`Please login before viewing this page.`}
                    </Box>
                </Modal>
            </UnauthenticatedTemplate>
        </>
    )
}
{% endraw %}
```

We made use of the `AuthenticatedTemplate` and `UnauthenticatedTemplate` components from the `msal-react` package to conditionally render components based on the authentication status of the current user. The `AuthenticatedTemplate` and `UnauthenticatedTemplate` components will only render their children if a user is authenticated or unauthenticated, respectively.  

## Create the page layout

We need to create a navigation bar to conditionally the sign in or sign out buttons as well as a link to another page with restricted content.  

Create a new directory called `Layout` inside the `Components` directory. Create a new file called `PageLayout.tsx` and add the following code snippet to it:  

```ts
{% raw %}
/*
 * Copyright (c) Microsoft Corporation. All rights reserved.
 * Licensed under the MIT License.
 */

import React from "react";
import { useIsAuthenticated } from "@azure/msal-react";
import { SignInButton } from "../Authentication/SignInButton";
import { SignOutButton } from "../Authentication/SignOutButton";
import AppBar from '@mui/material/AppBar';
import Box from '@mui/material/Box';
import Toolbar from '@mui/material/Toolbar';
import Typography from '@mui/material/Typography';
import Container from '@mui/material/Container'
import Button from "@mui/material/Button";
import Link from "next/link";

/**
 * Renders the navbar component with a sign in or sign out button depending on whether or not a user is authenticated
 * @param props
 */
interface  PageLayoutProps {
    children?: React.ReactNode
}

export const PageLayout = (props: PageLayoutProps) => {
    const isAuthenticated = useIsAuthenticated();
    const [anchorElNav, setAnchorElNav] = React.useState<EventTarget | null>(null)

    const handleCloseNavMenu = () => {
        setAnchorElNav(null)
    }

    return (
        <>
            <AppBar position="static">
                <Container maxWidth="xl">
                    <Toolbar disableGutters>
                        <Typography variant="h6" component={Link} href="/" display='flex' sx={{mr: 2}} >
                            Microsoft Identity Platform
                        </Typography>
                        <Box>
                            <Button
                                key={"Page 1"}
                                onClick={handleCloseNavMenu}
                                component={Link}
                                href={"/page_1"}
                                sx={{my: 2, color: 'white', display: 'block'}}
                            >
                                Page 1
                            </Button>
                        </Box>
                        <Box
                            sx={{
                                marginLeft: "auto",
                            }}
                        >
                            {isAuthenticated ? <SignOutButton /> : <SignInButton />}
                        </Box>
                    </Toolbar>
                </Container>
            </AppBar>
            <br />
            {props.children}
        </>
    );
};
{% endraw %}s
```

## Create a new page with restricted content
Inside the `app` directory of your project, create a new directory called `page_1` with an `page.tsx` file inside it. 

Add the following code to `page_1/page.tsx`:
```ts
'use client';

import Typography from '@mui/material/Typography'
import {useEffect} from "react";
import Container from "@mui/material/Container";
import Box from "@mui/material/Box";
import styles from '../../styles/Home.module.css'
import PermissionGate from "../../Components/PermissionGate/PermissionGate";

export default function Home() {
    useEffect(() => {
        document.title = 'Page 1'
    }, [])

    return (
        <>
            <main className={styles.container}>
                <Container>
                    <PermissionGate>
                        <Box
                            display="flex"
                            justifyContent="center"
                            alignItems="center">
                            <Typography variant="h1">Page 1</Typography>
                        </Box>
                    </PermissionGate>
                </Container>
            </main>
        </>
    )
}
```

Notice that the components inside "Page 1" are wrapped with the `PermissionGate` components, which means that they are only viewable if the user is authenticated.

## Create the Microsoft Graph client helper

To allow the SPA to request access to Microsoft Graph, a reference to the `graphConfig` object needs to be added. We will create a new `graph.ts` file that contains the Graph REST API endpoint defined in the `authConfig.ts` file.

In the `config` directory of your project, create `graph.ts` and add the following code snippet to request access to Microsoft Graph:  

```ts
import { graphConfig } from "./authConfig";

/**
 * Attaches a given access token to a MS Graph API call. Returns information about the user
 * @param accessToken
 */
export async function callMsGraph(accessToken: string) {
    const headers = new Headers();
    const bearer = `Bearer ${accessToken}`;

    headers.append("Authorization", bearer);

    const options = {
        method: "GET",
        headers: headers
    };

    return fetch(graphConfig.graphMeEndpoint, options)
        .then(response => response.json())
        .catch(error => console.log(error));
}
```

## Modify `page.tsx`

Open the `app/page.tsx` file and replace its content with the following code snippet:  

```ts
'use client';

import type {NextPage} from 'next'
import Head from 'next/head'
import styles from '../styles/Home.module.css'
import {AuthenticatedTemplate, UnauthenticatedTemplate, useMsal} from "@azure/msal-react";
import React from "react";
import {loginRequest, msalConfig} from "../config/authConfig";
import {callMsGraph} from "../config/graph";
import {ProfileData} from "../Components/Authentication/ProfileData";
import Button from "@mui/material/Button";
import Box from "@mui/material/Box";

const Home: NextPage = () => {
    /**
     * Renders information about the signed-in user or a button to retrieve data about the user
     */
    const ProfileContent = () => {
        const { instance, accounts } = useMsal();
        const [graphData, setGraphData] = React.useState(null);

        function RequestProfileData() {
            // Silently acquires an access token which is then attached to a request for MS Graph data
            instance
                .acquireTokenSilent({
                    ...loginRequest,
                    account: accounts[0],
                })
                .then((response) => {
                    callMsGraph(response.accessToken).then((response) => setGraphData(response));
                });
        }

        return (
            <>
                <h5 className="card-title">Welcome  {accounts[0] ? accounts[0].name: ''}</h5>
                <br/>
                {graphData ? (
                    <ProfileData graphData={graphData} />
                ) : (
                    <Button onClick={RequestProfileData}>
                        Request Profile Information
                    </Button>
                )}
            </>
        );
    };

    /**
     * If a user is authenticated the ProfileContent component above is rendered. Otherwise a message indicating a user is not authenticated is rendered.
     */
    const MainContent = () => {
        return (
            <div className="App">
                <h5>
                    <center>
                        Welcome to the Microsoft Authentication Library For TypeScript -
                        React/MUI SPA Tutorial
                    </center>
                </h5>
                <br />
                <AuthenticatedTemplate>
                    <ProfileContent />
                </AuthenticatedTemplate>

                <UnauthenticatedTemplate>
                    <h5>
                        <center>
                            Please sign-in to see your profile information.
                        </center>
                    </h5>
                </UnauthenticatedTemplate>
            </div>
        );
    };
    return (
        <div className={styles.container}>
            <Head>
                <title>MacID Authentication Example</title>
                <meta
                    name="description"
                    content="Generated by create next app"
                />
                <link rel="icon" href="/favicon.ico" />
            </Head>

            <main className={styles.main}>
                <Box>
                    <MainContent />
                </Box>
            </main>
        </div>
    )
}

export default Home
```

We added a `ProfileContent` function that is used to render the user's profile information. The `ProfileContent` component above is only rendered if the user is successfully authenticated since it is wrapped in an instance of `AuthenticatedTemplate`. Otherwise, a message indicating a user is not authenticated is rendered.  

## Sending Tokens to The Backend

You can use headers or secure cookies to send tokens from the frontend to the backend. Tokens can be used to validate the user identity in the backend. Never send tokens or sensitive information in query parameters.