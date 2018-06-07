using System;
using Labs.Utility;
using OpenTK;
using OpenTK.Graphics;
using OpenTK.Graphics.OpenGL;
using System.Drawing;
using System.Drawing.Imaging;

namespace Labs.ACW
{
    public class ACWWindow : GameWindow
    {

        public ACWWindow()
            : base(
                800, // Width
                600, // Height
                GraphicsMode.Default,
                "Assessed Coursework",
                GameWindowFlags.Default,
                DisplayDevice.Default,
                3, // major
                3, // minor
                GraphicsContextFlags.ForwardCompatible
                )
        {
        }

        //Declerations
        private int[] mVBO_IDs = new int[10];
        private int[] mVAO_IDs = new int[5];
        private int mTexture_ID;
        private ShaderUtility mShader;
        private ModelUtility middleCube, endCube, sphere, cylinder, topCube;
        private Matrix4 mView;
        private Vector3[] oldRedSpherePosition = new Vector3[100];
        private Vector3[] oldGreenSpherePosition = new Vector3[100];
        private float[] CylinderRadius = new float[6];
        private string[] CylinderType = new string[6];
        private Matrix4[] CylinderMatrix = new Matrix4[6];
        private Matrix4[] RedSphereMatrix = new Matrix4[100];
        private Matrix4[] GreenSphereMatrix = new Matrix4[100];
        private Vector3[] mRedSpherePosition = new Vector3[100];
        private Vector3[] mRedSphereVelocity = new Vector3[100];
        private Vector3[] mGreenSpherePosition = new Vector3[100];
        private Vector3[] mGreenSphereVelocity = new Vector3[100];
        private Vector3 accelerationDueToGravity = new Vector3(0, -0.981f, 0);
        private int numOfRedBalls, numOfGreenBalls;
        float seconds = 0;
        Random random = new Random();

        string cameraMode = "user";
        Timer mTimer;

        protected override void OnLoad(EventArgs e)
        {
            // Set some GL state
            GL.ClearColor(Color4.DodgerBlue);
            GL.Enable(EnableCap.DepthTest);
            GL.Enable(EnableCap.CullFace);

            //Loading stuff
            middleCube = ModelUtility.LoadModel(@"Utility/Models/cube_middle.sjg");
            endCube = ModelUtility.LoadModel(@"Utility/Models/cube_end.sjg");
            topCube = ModelUtility.LoadModel(@"Utility/Models/cube_top.sjg");
            sphere = ModelUtility.LoadModel(@"Utility/Models/sphere.bin");
            cylinder = ModelUtility.LoadModel(@"Utility/Models/cylinder.bin");
            mShader = new ShaderUtility(@"ACW/Shaders/VertexShader.vert", @"ACW/Shaders/FragmentShader.frag");
            mView = Matrix4.CreateTranslation(0, -0.3f, -1f);
            GL.UseProgram(mShader.ShaderProgramID);
            int vPositionLocation = GL.GetAttribLocation(mShader.ShaderProgramID, "vPosition");
            int vNormal = GL.GetAttribLocation(mShader.ShaderProgramID, "vNormal");
            int uView = GL.GetUniformLocation(mShader.ShaderProgramID, "uView");
            GL.UniformMatrix4(uView, true, ref mView);
            int uProjectionLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uProjection");
            Matrix4 projection = Matrix4.CreatePerspectiveFieldOfView(1, (float)ClientRectangle.Width / ClientRectangle.Height, 0.001f, 5);
            GL.UniformMatrix4(uProjectionLocation, true, ref projection);
            for (int i = 0; i < 100; ++i)
            {
                float x = random.Next(0, 180);
                x = (x-90) / 1000;
                float y = random.Next(0, 180);
                y = (y-90) / 1000;
                mRedSpherePosition[i] = new Vector3(x, 0.6f, y);
                oldRedSpherePosition[i] = mRedSpherePosition[i];
                mRedSphereVelocity[i] = new Vector3(0, 0, 0);
            }
            for (int i = 0; i < 100; ++i)
            {
                float x = random.Next(0, 180);
                x = (x - 90) / 1000;
                float y = random.Next(0, 180);
                y = (y - 90) / 1000;
                mGreenSpherePosition[i] = new Vector3(x, 0.6f, y);
                oldGreenSpherePosition[i] = mGreenSpherePosition[i];
                mGreenSphereVelocity[i] = new Vector3(0, 0, 0);
            }
            mTimer = new Timer();
            mTimer.Start();

            string filepath = @"ACW/texture.jpg";
            if (System.IO.File.Exists(filepath))
            {
                Bitmap TextureBitmap = new Bitmap(filepath);
                BitmapData TextureData = TextureBitmap.LockBits(
                new System.Drawing.Rectangle(0, 0, TextureBitmap.Width,
                TextureBitmap.Height), ImageLockMode.ReadOnly,
                System.Drawing.Imaging.PixelFormat.Format32bppRgb);
                GL.ActiveTexture(TextureUnit.Texture0);
                GL.GenTextures(1, out mTexture_ID);
                GL.BindTexture(TextureTarget.Texture2D, mTexture_ID);
                GL.TexImage2D(TextureTarget.Texture2D,
                0, PixelInternalFormat.Rgba, TextureData.Width, TextureData.Height,
                0, OpenTK.Graphics.OpenGL.PixelFormat.Bgra,
                PixelType.UnsignedByte, TextureData.Scan0);
                GL.TexParameter(TextureTarget.Texture2D, TextureParameterName.TextureMinFilter,
                (int)TextureMinFilter.Linear);
                GL.TexParameter(TextureTarget.Texture2D, TextureParameterName.TextureMagFilter,
                (int)TextureMagFilter.Linear);
                TextureBitmap.UnlockBits(TextureData);
            }
            else
            {
                throw new Exception("Could not find file " + filepath);
            }

            //Vertex arrays and buffers
            GL.GenVertexArrays(mVAO_IDs.Length, mVAO_IDs);
            GL.GenBuffers(mVBO_IDs.Length, mVBO_IDs);

            //middlecube
            GL.BindVertexArray(mVAO_IDs[0]);
            GL.BindBuffer(BufferTarget.ArrayBuffer, mVBO_IDs[0]);
            GL.BufferData(BufferTarget.ArrayBuffer, (IntPtr)(middleCube.Vertices.Length * sizeof(float)), middleCube.Vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, mVBO_IDs[1]);
            GL.BufferData(BufferTarget.ElementArrayBuffer, (IntPtr)(middleCube.Indices.Length * sizeof(float)), middleCube.Indices, BufferUsageHint.StaticDraw);
            int size;
            GL.GetBufferParameter(BufferTarget.ArrayBuffer, BufferParameterName.BufferSize, out size);
            if (middleCube.Vertices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Vertex data not loaded onto graphics card correctly");
            }

            GL.GetBufferParameter(BufferTarget.ElementArrayBuffer, BufferParameterName.BufferSize, out size);
            if (middleCube.Indices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Index data not loaded onto graphics card correctly");
            }
            GL.EnableVertexAttribArray(vPositionLocation);
            GL.VertexAttribPointer(vPositionLocation, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 0);
            GL.EnableVertexAttribArray(vNormal);
            GL.VertexAttribPointer(vNormal, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 3 * sizeof(float));
            GL.BindVertexArray(0);

            //endcube
            GL.BindVertexArray(mVAO_IDs[1]);
            GL.BindBuffer(BufferTarget.ArrayBuffer, mVBO_IDs[2]);
            GL.BufferData(BufferTarget.ArrayBuffer, (IntPtr)(endCube.Vertices.Length * sizeof(float)), endCube.Vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, mVBO_IDs[3]);
            GL.BufferData(BufferTarget.ElementArrayBuffer, (IntPtr)(endCube.Indices.Length * sizeof(float)), endCube.Indices, BufferUsageHint.StaticDraw);
            GL.GetBufferParameter(BufferTarget.ArrayBuffer, BufferParameterName.BufferSize, out size);
            if (endCube.Vertices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Vertex data not loaded onto graphics card correctly");
            }

            GL.GetBufferParameter(BufferTarget.ElementArrayBuffer, BufferParameterName.BufferSize, out size);
            if (endCube.Indices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Index data not loaded onto graphics card correctly");
            }
            GL.EnableVertexAttribArray(vPositionLocation);
            GL.VertexAttribPointer(vPositionLocation, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 0);
            GL.EnableVertexAttribArray(vNormal);
            GL.VertexAttribPointer(vNormal, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 3 * sizeof(float));
            GL.BindVertexArray(0);

            //sphere
            GL.BindVertexArray(mVAO_IDs[2]);
            GL.BindBuffer(BufferTarget.ArrayBuffer, mVBO_IDs[4]);
            GL.BufferData(BufferTarget.ArrayBuffer, (IntPtr)(sphere.Vertices.Length * sizeof(float)), sphere.Vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, mVBO_IDs[5]);
            GL.BufferData(BufferTarget.ElementArrayBuffer, (IntPtr)(sphere.Indices.Length * sizeof(float)), sphere.Indices, BufferUsageHint.StaticDraw);
            GL.GetBufferParameter(BufferTarget.ArrayBuffer, BufferParameterName.BufferSize, out size);
            if (sphere.Vertices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Vertex data not loaded onto graphics card correctly");
            }

            GL.GetBufferParameter(BufferTarget.ElementArrayBuffer, BufferParameterName.BufferSize, out size);
            if (sphere.Indices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Index data not loaded onto graphics card correctly");
            }
            GL.EnableVertexAttribArray(vPositionLocation);
            GL.VertexAttribPointer(vPositionLocation, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 0);
            GL.EnableVertexAttribArray(vNormal);
            GL.VertexAttribPointer(vNormal, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 3 * sizeof(float));
            GL.BindVertexArray(0);

            //cylinder
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.BindBuffer(BufferTarget.ArrayBuffer, mVBO_IDs[6]);
            GL.BufferData(BufferTarget.ArrayBuffer, (IntPtr)(cylinder.Vertices.Length * sizeof(float)), cylinder.Vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, mVBO_IDs[7]);
            GL.BufferData(BufferTarget.ElementArrayBuffer, (IntPtr)(cylinder.Indices.Length * sizeof(float)), cylinder.Indices, BufferUsageHint.StaticDraw);
            GL.GetBufferParameter(BufferTarget.ArrayBuffer, BufferParameterName.BufferSize, out size);
            if (cylinder.Vertices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Vertex data not loaded onto graphics card correctly");
            }

            GL.GetBufferParameter(BufferTarget.ElementArrayBuffer, BufferParameterName.BufferSize, out size);
            if (cylinder.Indices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Index data not loaded onto graphics card correctly");
            }
            GL.EnableVertexAttribArray(vPositionLocation);
            GL.VertexAttribPointer(vPositionLocation, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 0);
            GL.EnableVertexAttribArray(vNormal);
            GL.VertexAttribPointer(vNormal, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 3 * sizeof(float));
            GL.BindVertexArray(0);

            //topcube
            GL.BindVertexArray(mVAO_IDs[4]);
            GL.BindBuffer(BufferTarget.ArrayBuffer, mVBO_IDs[8]);
            GL.BufferData(BufferTarget.ArrayBuffer, (IntPtr)(topCube.Vertices.Length * sizeof(float)), topCube.Vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, mVBO_IDs[9]);
            GL.BufferData(BufferTarget.ElementArrayBuffer, (IntPtr)(topCube.Indices.Length * sizeof(float)), topCube.Indices, BufferUsageHint.StaticDraw);
            GL.GetBufferParameter(BufferTarget.ArrayBuffer, BufferParameterName.BufferSize, out size);
            if (topCube.Vertices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Vertex data not loaded onto graphics card correctly");
            }

            GL.GetBufferParameter(BufferTarget.ElementArrayBuffer, BufferParameterName.BufferSize, out size);
            if (topCube.Indices.Length * sizeof(float) != size)
            {
                throw new ApplicationException("Index data not loaded onto graphics card correctly");
            }
            GL.EnableVertexAttribArray(vPositionLocation);
            GL.VertexAttribPointer(vPositionLocation, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 0);
            GL.EnableVertexAttribArray(vNormal);
            GL.VertexAttribPointer(vNormal, 3, VertexAttribPointerType.Float, true, 6 * sizeof(float), 3 * sizeof(float));
            GL.BindVertexArray(0);

            //Eye Position
            int uEyePositionLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uEyePosition");
            Vector4 eyePosition = Vector4.Transform(new Vector4(0,0,0,0), mView);
            GL.Uniform4(uEyePositionLocation, ref eyePosition);

            int uLightPositionLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uLight.Position");
            Vector4 lightPosition = new Vector4(4, -4, 2f, 1);
            lightPosition = Vector4.Transform(lightPosition, mView);
            GL.Uniform4(uLightPositionLocation, lightPosition);
            int uAmbientLightLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uLight.AmbientLight");
            Vector3 colour = new Vector3(1.0f, 1.0f, 0.90f);
            GL.Uniform3(uAmbientLightLocation, colour);

            int uDiffuseLightLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uLight.DiffuseLight");
            colour = new Vector3(1.0f, 1.0f, 0.90f);
            GL.Uniform3(uDiffuseLightLocation, colour);

            int uSpecularLightLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uLight.SpecularLight");
            colour = new Vector3(1.0f, 1.0f, 0.90f);
            GL.Uniform3(uSpecularLightLocation, colour);

            int uAmbientReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.AmbientReflectivity");
            colour = new Vector3(0.2f, 0.2f, 0.2f);
            GL.Uniform3(uAmbientReflectivityLocation, colour);

            int uDiffuseReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.DiffuseReflectivity");
            GL.Uniform3(uDiffuseReflectivityLocation, colour);

            int uSpecularReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.SpecularReflectivity");
            GL.Uniform3(uSpecularReflectivityLocation, colour);

            int uShininessLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.Shininess");
            float Shininess = 30;
            GL.Uniform1(uShininessLocation, Shininess);


            base.OnLoad(e);
        }

        protected override void OnKeyPress(KeyPressEventArgs e)
        {
            base.OnKeyPress(e);
            if (e.KeyChar == 'v')
            {
                cameraMode = "fixed";
                mView = Matrix4.CreateTranslation(0, -0.3f, -1f);
                MoveCamera();
            }
            if (e.KeyChar == 'u')
            {
                cameraMode = "user";
                mView = Matrix4.CreateTranslation(0, -0.3f, -1f);
                MoveCamera();
            }
            if (e.KeyChar == 'p')
            {
                cameraMode = "path";
                mView = Matrix4.CreateTranslation(0, -0.3f, -1f);
                MoveCamera();
            }
            if (e.KeyChar == 'a' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(0.01f, 0, 0);
                MoveCamera();
            }            if (e.KeyChar == 'd' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(-0.01f, 0, 0);
                MoveCamera();
            }            if (e.KeyChar == 's' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(0, 0, -0.01f);
                MoveCamera();
            }            if (e.KeyChar == 'w' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(0, 0, 0.01f);
                MoveCamera();
            }            if (e.KeyChar == 'r' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(0, -0.01f, 0);
                MoveCamera();
            }            if (e.KeyChar == 'f' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateTranslation(0, 0.01f, 0);
                MoveCamera();
            }            if (e.KeyChar == 'e' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateRotationY(0.01f);
                MoveCamera();
            }            if (e.KeyChar == 'q' && cameraMode == "user")
            {
                mView = mView * Matrix4.CreateRotationY(-0.01f);
                MoveCamera();
            }
        }

        private void MoveCamera()
        {
            int uView = GL.GetUniformLocation(mShader.ShaderProgramID, "uView");
            GL.UniformMatrix4(uView, true, ref mView);

            int uLightPositionLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uLight.Position");
            Vector4 lightPosition = new Vector4(4, -4, 2f, 1);
            lightPosition = Vector4.Transform(lightPosition, mView);
            GL.Uniform4(uLightPositionLocation, lightPosition);

            int uEyePositionLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uEyePosition");
            Vector4 eyePosition = Vector4.Transform(new Vector4(0, 0, 0, 0), mView);
            GL.Uniform4(uEyePositionLocation, ref eyePosition);
        }

        protected override void OnResize(EventArgs e)
        {
            base.OnResize(e);
            Matrix4 projection = Matrix4.CreatePerspectiveFieldOfView(1, (float)ClientRectangle.Width / ClientRectangle.Height, 0.5f, 5);
            GL.Viewport(this.ClientRectangle);
        }

        protected override void OnUpdateFrame(FrameEventArgs e)
        {
 	        base.OnUpdateFrame(e);

            if (cameraMode == "path")
            {
                mView = mView * Matrix4.CreateRotationY(0.01f) * Matrix4.CreateTranslation(0.01f, 0, 0);
                MoveCamera();
            }

            float timestep = mTimer.GetElapsedSeconds();

            seconds = seconds + timestep;

            if(seconds > 0.25 && numOfRedBalls < 100)
            {
                if (numOfGreenBalls > numOfRedBalls)
                {
                    ++numOfRedBalls;
                }
                else
                {
                    ++numOfGreenBalls;
                }
                seconds = 0;
            }
            for (int a = 0; a < numOfRedBalls; ++a)
            {
                mRedSphereVelocity[a] = mRedSphereVelocity[a] + accelerationDueToGravity * timestep;
                mRedSpherePosition[a] = mRedSpherePosition[a] + mRedSphereVelocity[a] * timestep;
                if (mRedSpherePosition[a].Y < (-0.1 + 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].Y = (-mRedSphereVelocity[a].Y / 10) * 6;
                }
                if (mRedSpherePosition[a].Y > (0.7 - 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].Y = (-mRedSphereVelocity[a].Y / 10) * 6;
                }
                if (mRedSpherePosition[a].X > (0.1 - 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].X = (-mRedSphereVelocity[a].X / 10) * 6;
                }
                if (mRedSpherePosition[a].X < (-0.1 + 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].X = (-mRedSphereVelocity[a].X / 10) * 6;
                }
                if (mRedSpherePosition[a].Z > (0.1 - 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].Z = (-mRedSphereVelocity[a].Z / 10) * 6;
                }
                if (mRedSpherePosition[a].Z < (-0.1 + 0.007))
                {
                    mRedSpherePosition[a] = oldRedSpherePosition[a];
                    mRedSphereVelocity[a].Z = (-mRedSphereVelocity[a].Z / 10) * 6;
                }
                for(int i = 0; i < numOfRedBalls; ++i)
                {
                    if (i == a)
                    {
                    }
                    else if ((RedSphereMatrix[a].ExtractTranslation().Xzy - RedSphereMatrix[i].ExtractTranslation().Xzy).Length <  0.007f + 0.007f)
                    {
                        Vector3 normal = (RedSphereMatrix[i].ExtractTranslation().Xzy - RedSphereMatrix[a].ExtractTranslation().Xzy).Normalized();
                        mRedSphereVelocity[a] = mRedSphereVelocity[a] - Vector3.Dot(normal, mRedSphereVelocity[a]) * normal;
                        mRedSphereVelocity[i] = mRedSphereVelocity[i] - Vector3.Dot(normal, mRedSphereVelocity[i]) * normal;
                    }
                }
                oldRedSpherePosition[a] = mRedSpherePosition[a];
            }
            for (int a = 0; a < numOfGreenBalls; ++a)
            {
                mGreenSphereVelocity[a] = mGreenSphereVelocity[a] + accelerationDueToGravity * timestep;
                mGreenSpherePosition[a] = mGreenSpherePosition[a] + mGreenSphereVelocity[a] * timestep;
                if (mGreenSpherePosition[a].Y < (-0.1 + 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].Y = (-mGreenSphereVelocity[a].Y / 10) * 4;
                }
                if (mGreenSpherePosition[a].Y > (0.7 - 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].Y = (-mGreenSphereVelocity[a].Y / 10) * 4;
                }
                if (mGreenSpherePosition[a].X > (0.1 - 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].X = (-mGreenSphereVelocity[a].X / 10) * 4;
                }
                if (mGreenSpherePosition[a].X < (-0.1 + 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].X = (-mGreenSphereVelocity[a].X / 10) * 4;
                }
                if (mGreenSpherePosition[a].Z > (0.1 - 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].Z = (-mGreenSphereVelocity[a].Z / 10) * 4;
                }
                if (mGreenSpherePosition[a].Z < (-0.1 + 0.004))
                {
                    mGreenSpherePosition[a] = oldGreenSpherePosition[a];
                    mGreenSphereVelocity[a].Z = (-mGreenSphereVelocity[a].Z / 10) * 4;
                }
                for (int i = 0; i < numOfGreenBalls; ++i)
                {
                    if (i == a)
                    {
                    }
                    else if ((GreenSphereMatrix[a].ExtractTranslation().Xzy - GreenSphereMatrix[i].ExtractTranslation().Xzy).Length < 0.004f + 0.004f)
                    {
                        Vector3 normal = (GreenSphereMatrix[i].ExtractTranslation().Xzy - GreenSphereMatrix[a].ExtractTranslation().Xzy).Normalized();
                        mGreenSphereVelocity[a] = mGreenSphereVelocity[a] - Vector3.Dot(normal, mGreenSphereVelocity[a]) * normal;
                        mGreenSphereVelocity[i] = mGreenSphereVelocity[i] - Vector3.Dot(normal, mGreenSphereVelocity[i]) * normal;
                    }
                }
                oldGreenSpherePosition[a] = mGreenSpherePosition[a];
            }
        }

        protected override void OnRenderFrame(FrameEventArgs e)
        {
            base.OnRenderFrame(e);
            GL.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

            //Bottom Cube
            int uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            Matrix4 m1 = Matrix4.CreateTranslation(0, 0, 0);
            GL.UniformMatrix4(uModelLocation, true, ref m1);
            GL.BindVertexArray(mVAO_IDs[1]);
            GL.DrawElements(BeginMode.Triangles, endCube.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Top Cube
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            m1 = Matrix4.CreateTranslation(0.0f, 0.6f, 0);
            GL.UniformMatrix4(uModelLocation, true, ref m1);
            GL.BindVertexArray(mVAO_IDs[4]);
            GL.DrawElements(BeginMode.Triangles, topCube.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Middle Cube
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            m1 = Matrix4.CreateTranslation(0.0f, 0.4f, 0);
            GL.UniformMatrix4(uModelLocation, true, ref m1);
            GL.BindVertexArray(mVAO_IDs[0]);
            GL.DrawElements(BeginMode.Triangles, middleCube.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Middle Cube
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            m1 = Matrix4.CreateTranslation(0.0f, 0.2f, 0);
            GL.UniformMatrix4(uModelLocation, true, ref m1);
            GL.BindVertexArray(mVAO_IDs[0]);
            GL.DrawElements(BeginMode.Triangles, middleCube.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Animated Sphere - Red
            int uAmbientReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.AmbientReflectivity");
            Vector3 colour = new Vector3(0.8627451f, 0.078431375f, 0.23529412f);
            GL.Uniform3(uAmbientReflectivityLocation, colour);

            int uDiffuseReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.DiffuseReflectivity");
            GL.Uniform3(uDiffuseReflectivityLocation, colour);

            int uSpecularReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.SpecularReflectivity");
            GL.Uniform3(uSpecularReflectivityLocation, colour);

            int uShininessLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.Shininess");
            float Shininess = 0;
            GL.Uniform1(uShininessLocation, Shininess);
            for (int a = 0; a < numOfRedBalls; ++a)
            {
                uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
                RedSphereMatrix[a] = Matrix4.CreateScale(0.007f) * Matrix4.CreateTranslation(mRedSpherePosition[a]);
                GL.UniformMatrix4(uModelLocation, true, ref RedSphereMatrix[a]);
                GL.BindVertexArray(mVAO_IDs[2]);
                GL.DrawElements(BeginMode.Triangles, sphere.Indices.Length, DrawElementsType.UnsignedInt, 0);
            }

            //Animated Sphere - Green
            uAmbientReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.AmbientReflectivity");
            colour = new Vector3(0.137255f, 0.556863f, 0.137255f);
            GL.Uniform3(uAmbientReflectivityLocation, colour);

            uDiffuseReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.DiffuseReflectivity");
            GL.Uniform3(uDiffuseReflectivityLocation, colour);

            uSpecularReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.SpecularReflectivity");
            GL.Uniform3(uSpecularReflectivityLocation, colour);

            uShininessLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.Shininess");
            Shininess = 30;
            GL.Uniform1(uShininessLocation, Shininess);

            for (int a = 0; a < numOfGreenBalls; ++a)
            {
                uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
                GreenSphereMatrix[a] = Matrix4.CreateScale(0.004f) * Matrix4.CreateTranslation(mGreenSpherePosition[a]);
                GL.UniformMatrix4(uModelLocation, true, ref GreenSphereMatrix[a]);
                GL.BindVertexArray(mVAO_IDs[2]);
                GL.DrawElements(BeginMode.Triangles, sphere.Indices.Length, DrawElementsType.UnsignedInt, 0);
            }

            //Sphere o' Doom
            uAmbientReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.AmbientReflectivity");
            colour = new Vector3(0.11764706f, 0.5647059f, 1.0f);
            GL.Uniform3(uAmbientReflectivityLocation, colour);

            uDiffuseReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.DiffuseReflectivity");
            GL.Uniform3(uDiffuseReflectivityLocation, colour);

            uSpecularReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.SpecularReflectivity");
            GL.Uniform3(uSpecularReflectivityLocation, colour);

            uShininessLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.Shininess");
            Shininess = 30;
            GL.Uniform1(uShininessLocation, Shininess);

            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            m1 = Matrix4.CreateTranslation(0, 0, 0);
            Matrix4 s1 = Matrix4.CreateScale(0.035f);
            Matrix4 c1 = s1 * m1;
            GL.UniformMatrix4(uModelLocation, true, ref c1);
            GL.BindVertexArray(mVAO_IDs[2]);
            GL.DrawElements(BeginMode.Triangles, sphere.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uAmbientReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.AmbientReflectivity");
            colour = new Vector3(0.2f, 0.2f, 0.2f);
            GL.Uniform3(uAmbientReflectivityLocation, colour);

            uDiffuseReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.DiffuseReflectivity");
            GL.Uniform3(uDiffuseReflectivityLocation, colour);

            uSpecularReflectivityLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uMaterial.SpecularReflectivity");
            GL.Uniform3(uSpecularReflectivityLocation, colour);

            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[0] = 0.015f;
            CylinderType[0] = "nondiagonal";
            m1 = Matrix4.CreateTranslation(0, 0.4f, 0);
            s1 = Matrix4.CreateScale(0.1f,0.015f,0.015f);
            Matrix4 r1 = Matrix4.CreateRotationZ(1.5708f);
            CylinderMatrix[0] = r1 * s1 * m1 ;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[0]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[1] = 0.0075f;
            CylinderType[1] = "nondiagonal";
            m1 = Matrix4.CreateTranslation(0, 0.445f, 0);
            s1 = Matrix4.CreateScale(0.0075f, 0.0075f, 0.1f);
            r1 = Matrix4.CreateRotationX(1.5708f);
            CylinderMatrix[1] = r1 * s1 * m1;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[1]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[2] = 0.0075f;
            CylinderType[2] = "nondiagonal";
            m1 = Matrix4.CreateTranslation(-0.050f, 0.355f, 0);
            s1 = Matrix4.CreateScale(0.0075f, 0.0075f, 0.1f);
            r1 = Matrix4.CreateRotationX(1.5708f);
            CylinderMatrix[2] = r1 * s1 * m1;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[2]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[3] = 0.0075f;
            CylinderType[3] = "nondiagonal";
            m1 = Matrix4.CreateTranslation(0.050f, 0.355f, 0f);
            s1 = Matrix4.CreateScale(0.0075f, 0.0075f, 0.1f);
            r1 = Matrix4.CreateRotationX(1.5708f);
            CylinderMatrix[3] = r1 * s1 * m1;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[3]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[4] = 0.015f;
            CylinderType[4] = "diagonal";
            m1 = Matrix4.CreateTranslation(0, 0.25f, 0);
            s1 = Matrix4.CreateScale(0.015f, 0.1f, 0.015f);
            r1 = Matrix4.CreateRotationZ(1.5708f);
            Matrix4 r2 = Matrix4.CreateRotationY(0.785398f);
            CylinderMatrix[4] = s1 * r1 * r2 * m1;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[4]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            //Cylinder
            uModelLocation = GL.GetUniformLocation(mShader.ShaderProgramID, "uModel");
            CylinderRadius[5] = 0.015f;
            CylinderType[5] = "diagonal";
            m1 = Matrix4.CreateTranslation(0, 0.15f, 0);
            s1 = Matrix4.CreateScale(0.015f, 0.1f, 0.015f);
            r1 = Matrix4.CreateRotationZ(1.5708f);
            r2 = Matrix4.CreateRotationY(-0.785398f);
            CylinderMatrix[5] = s1 *r1* r2* m1;
            GL.UniformMatrix4(uModelLocation, true, ref CylinderMatrix[5]);
            GL.BindVertexArray(mVAO_IDs[3]);
            GL.DrawElements(BeginMode.Triangles, cylinder.Indices.Length, DrawElementsType.UnsignedInt, 0);

            GL.BindVertexArray(0);
            this.SwapBuffers();
        }

        protected override void OnUnload(EventArgs e)
        {
            GL.BindBuffer(BufferTarget.ArrayBuffer, 0);
            GL.BindBuffer(BufferTarget.ElementArrayBuffer, 0);
            GL.BindVertexArray(0);
            GL.DeleteBuffers(mVBO_IDs.Length, mVBO_IDs);
            GL.DeleteVertexArrays(mVAO_IDs.Length, mVAO_IDs);
            mShader.Delete();
            base.OnUnload(e);
        }
    }
}
