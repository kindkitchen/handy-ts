# handy-ts
* ## Omit++
  ```ts
  /**
   * @description
   * 1. omit only property that is existed in original T
   * 2. make sure that the omitted property is not accidenticaly will present again
   * 
   * @example
   * 
   type PrivateUser = { id: number, email: string, password: string, }
   type PublicUser = OmitStrict<PrivateUser, 'password'>;
   
   const userWithPassword: PrivateUser = { id: 1, email: 'example@mail.com', password: 'qwerty'};
   const userWithSimpleOmit: Omit<PrivateUser, 'password'> = userWithPassword; // Ok... oops... password is still there
   const user: PublicUser = userWithPassword; // Error!
   */
  export type OmitStrict<
    T extends Record<string, any>,
    K extends keyof T,
  > = Omit<T, K> & Partial<Record<K, never>>
  
  
  /**
   * @description omit only property that is exist in T and replace it with your own
   */
  export type OmitReplace<
    T extends Record<string, any>,
    U extends Partial<Record<keyof T, any>>,
  > = Omit<T, keyof U> & U;
  
  /**
   * @description add your property to T (replace previous one if exist)
   */
  export type AddOrReplace<
    T extends Record<string, any>,
    U extends Record<string, any>
  > = Omit<T, keyof U> & U;

  ```
* ## Arr++
  ```ts
  /**
  * @description rm first only element from given array
  */
  type Tail<T extends any[]> = T extends [infer _first, ...infer Rest] ? Rest : never;
  ```

* ## Help with graphql resolvers
  ```ts
    /**
   * @description
   * This type result to aka `Omit<First, keyof Second>`.
   * ---
   * > It's purpose is to be sure that we omit only keys that are represent relation and so we prove it by providing resolver, that worry about this field
   * @example
   * #### Pseudo-code example:
   * Suppose we have `User.posts` and of course `posts` this is relation and it will be computed dynamically if client ask about this property. So it will be not resolved directly here. So we can user this type and omit `posts` property by providing resolver that really worry about this property.
   *
   * #### Aka implementation of the example:
   * ```graphql
   * type User {
   *   _id: ID
   *   name: String!
   *   posts: [Post]!
   * }
   *
   * type Post {
   *   _id: ID
   *   title: String!
   *   description: String
   * }
   *
   * type Query {
   *   get_author(name: String): User
   * }
   *
   * ```
   * So in our code we can so something like:
   * ```typescript
   * type UserResolver = {
   *   /// ...
   *   posts: () => Promise<Post[]> /// <= so this is real place where we can obtain posts
   * }
   *
   * type Query = {
   *   get_author: (name: string) => Promise<
   *     /// HERE!
   *     SkipRelation<User, {
   *       posts: UserResolver /// <= we omit posts but prove that we know that resolver is exists for it
   *     }>
   *   >
   * }
   *
   * ```
   *
   * ---
   * #### How it works?
   * 1. check that we omit keys from First but not any ones (keyof Second)
   * 2. check that we know some type, that contains omitted value (Second[keyof Second])
   */
  export type SkipRelations<
    T,
    R extends Partial<
      {
        [K in keyof T]: Promise<T[K]> | T[K] | (() => T[K] | Promise<T[K]>);
      }
    >,
  > = Omit<T, keyof R>;
  ```

##### _cp all_
```ts
declare global {
  type Tail<T extends any[]> = T extends [infer _first, ...infer Rest] ? Rest : never;
  type OmitStrict<
    T extends Record<string, any>,
    K extends keyof T,
  > = Omit<T, K> & Partial<Record<K, never>>
  type OmitReplace<
    T extends Record<string, any>,
    U extends Partial<Record<keyof T, any>>,
  > = Omit<T, keyof U> & U;
  
  type AddOrReplace<
    T extends Record<string, any>,
    U extends Record<string, any>
  > = Omit<T, keyof U> & U;
  
  type SkipRelations<
    T,
    R extends Partial<
      {
        [K in keyof T]: Promise<T[K]> | T[K] | (() => T[K] | Promise<T[K]>);
      }
    >,
    > = Omit<T, keyof R>;
  }

export { };
```
